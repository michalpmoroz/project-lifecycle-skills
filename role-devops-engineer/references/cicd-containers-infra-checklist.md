# CI/CD, Containers & Infrastructure Reliability Checklist

> **Baseline Versions**: GitHub Actions (Ubuntu 24.04), Docker 27.x / BuildKit, Docker Compose v2, OpenTofu 1.8+ / Terraform 1.9+, Cloudflare Pages & Workers (`wrangler` 3.x), Linux Kernel 6.x (Ubuntu 24.04 LTS / Debian 12).

> [!NOTE]
> **Continuous Evolution Mandate**: This checklist is a baseline floor. Continuously refine pipeline speed, supply-chain attestations, container minimalism, and disaster recovery automation as project requirements scale.

---

## 1. GitHub Actions CI/CD Hardening & Optimization

### 1.1 Supply-Chain Security & Least Privilege
- [ ] **Pin Actions to Full 40-Character Commit SHA**:
  - Never use mutable tags like `@v4` or `@main` for third-party actions in production workflows.
  - Format: `uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2`.
- [ ] **Restrict Default `GITHUB_TOKEN` Permissions**:
  - Declare `permissions: contents: read` at the top level of every workflow file.
  - Grant write scopes (`id-token: write`, `packages: write`, `pull-requests: write`) only at the specific job that requires them.
- [ ] **Eliminate Script Injection Vulnerabilities**:
  - Never interpolate untrusted inputs (`${{ github.event.issue.title }}`, `${{ github.event.pull_request.body }}`, `${{ github.head_ref }}`) directly inside a `run:` shell script.
  - Bind them to environment variables first:
    ```yaml
    # ❌ VULNERABLE to arbitrary shell command execution via crafted PR title
    - run: echo "PR Title is ${{ github.event.pull_request.title }}"

    # ✅ SAFE: Passed via environment variable; shell does not expand subcommands
    - env:
        PR_TITLE: ${{ github.event.pull_request.title }}
      run: echo "PR Title is $PR_TITLE"
    ```
- [ ] **Passwordless OIDC Authentication over Static Secrets**:
  - Use OpenID Connect (`permissions: id-token: write`) to exchange short-lived tokens with AWS, GCP, Azure, or artifact registries instead of storing long-lived IAM keys in GitHub Secrets.
- [ ] **Safe `pull_request` vs. `pull_request_target` Usage**:
  - Never check out and execute untrusted fork PR code inside a `pull_request_target` workflow that has access to repository secrets or write permissions.

### 1.2 Pipeline Speed, Concurrency & Determinism
- [ ] **Cancel Superseded PR Runs (`concurrency`)**:
  - Configure concurrency groups on CI workflows, while keeping production deployments serialized (`cancel-in-progress: false` on `main`/deploy):
    ```yaml
    concurrency:
      group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
      cancel-in-progress: ${{ github.ref != 'refs/heads/main' }}
    ```
- [ ] **Explicit Job Timeouts**: Set `timeout-minutes: 15` (or tighter) on every job so a hung network call or deadlocked test never consumes the default 360-minute runner quota.
- [ ] **Frozen Lockfile Installs & Caching**:
  - Always use `npm ci`, `pnpm install --frozen-lockfile`, or `yarn install --immutable`.
  - Cache package manager stores (`~/.pnpm-store`, `~/.npm`) and build caches (Turbo / Nx / Next.js `.next/cache`, Playwright browser binaries) keyed by `hashFiles('**/pnpm-lock.yaml')`.
- [ ] **Matrix Sharding for E2E Tests**: Shard Playwright E2E suites across parallel workers (`strategy: matrix: shard: [1/4, 2/4, 3/4, 4/4]`) and merge HTML/blob reports in a downstream summary job.

---

## 2. Docker & Container Security / Optimization (Docker 27 / BuildKit)

### 2.1 `.dockerignore` Hygiene
- [ ] **Mandatory `.dockerignore`**: Prevent build context bloat and accidental secret leakage into image layers by excluding:
  ```dockerignore
  .git
  .github
  node_modules
  dist
  build
  .next
  coverage
  .env*
  !.env.example
  *.log
  docs
  tests
  playwright-report
  ```

### 2.2 Multi-Stage Production `Dockerfile` Pattern
- [ ] **Layer Caching Order**: Copy only package manifests (`package.json`, `pnpm-lock.yaml`) and install dependencies **before** copying application source code so code edits do not invalidate the dependency layer cache.
- [ ] **Minimal Base Images**: Use `node:22-alpine` (with pinned digest), `gcr.io/distroless/nodejs22-debian12:nonroot`, or Chainguard images (`cgr.dev/chainguard/node`) for the final runtime stage.
- [ ] **Non-Root User (`USER`) & PID 1 Signal Handling (`tini`)**:
  ```dockerfile
  # syntax=docker/dockerfile:1.9
  FROM node:22.12-alpine3.21 AS base
  RUN corepack enable && apk add --no-cache libc6-compat

  # Stage 1: Install all dependencies (for build)
  FROM base AS deps
  WORKDIR /app
  COPY package.json pnpm-lock.yaml ./
  RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --frozen-lockfile

  # Stage 2: Build application
  FROM base AS builder
  WORKDIR /app
  COPY --from=deps /app/node_modules ./node_modules
  COPY . .
  RUN pnpm run build && pnpm prune --prod

  # Stage 3: Hardened Minimal Production Runtime
  FROM node:22.12-alpine3.21 AS runner
  RUN apk add --no-cache tini
  WORKDIR /app
  ENV NODE_ENV=production

  # Create dedicated non-root user & group
  RUN addgroup --system --gid 1001 appgroup && \
      adduser --system --uid 1001 --ingroup appgroup appuser

  COPY --from=builder --chown=appuser:appgroup /app/package.json ./
  COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules
  COPY --from=builder --chown=appuser:appgroup /app/dist ./dist

  USER appuser
  EXPOSE 3000

  HEALTHCHECK --interval=15s --timeout=3s --start-period=10s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://127.0.0.1:3000/health/live || exit 1

  ENTRYPOINT ["/sbin/tini", "--"]
  CMD ["node", "dist/main.js"]
  ```
- [ ] **Never Pass Secrets via `ARG` or `ENV` in Build Stages**: `ARG` and `ENV` values persist in `docker history`. If build-time secrets are required (e.g., private registry token), use BuildKit secret mounts: `RUN --mount=type=secret,id=npmrc,target=/root/.npmrc ...`.
- [ ] **Container Runtime Hardening (`docker-compose.yml` / K8s)**:
  - Set `read_only: true` on the root filesystem (mounting `tmpfs: [/tmp]` if needed).
  - Set `security_opt: ["no-new-privileges:true"]` and `cap_drop: ["ALL"]`.
  - Enforce explicit memory and CPU limits (`deploy.resources.limits`) to prevent a memory leak in one container from triggering host OOM kills on database or proxy containers.

---

## 3. Continuous Deployment, Database Migrations & Reliability

### 3.1 Immutable Artifacts & Promotion
- [ ] **Build Once, Deploy Anywhere**: Compile and package the artifact/container once in CI, tag it with the immutable Git commit SHA (`ghcr.io/org/app:sha-a1b2c3d`), scan it with Trivy, and promote that exact image digest to Staging and Production.
- [ ] **Environment Configuration Parity**: Inject environment-specific settings exclusively via runtime environment variables validated at startup—never via branch-specific code paths or separate builds.

### 3.2 Zero-Downtime Rollout Strategies
- [ ] **Blue-Green or Rolling Deployment with Readiness Gates**:
  1. Start new container(s) (`Green`) alongside existing container(s) (`Blue`).
  2. Wait until `Green` passes `/health/ready` checks (verifying DB/cache connectivity).
  3. Switch reverse proxy (Caddy / Traefik / Cloudflare / Nginx) traffic to `Green`.
  4. Send `SIGTERM` to `Blue`, allowing in-flight requests to drain for `20–30s` before stopping.
- [ ] **Separate Liveness vs. Readiness Endpoints**:
  - `/health/live`: Lightweight process check (`200 OK` if event loop is responsive). Never check external DBs here, or a brief DB blip will cause orchestrators to restart all app pods simultaneously!
  - `/health/ready`: Deep readiness check verifying DB pool and critical local dependencies before accepting traffic.

### 3.3 Safe Database Migrations in CD Pipelines
- [ ] **Pre-Deploy Migration Step**: Execute `migrate deploy` as an isolated single-instance job **after** backup verification and **before** switching traffic to the new application instances.
- [ ] **Advisory Lock Serialization**: Ensure the migration runner acquires a PostgreSQL advisory lock (`pg_try_advisory_lock`) so concurrent deployment jobs can never run migrations simultaneously.
- [ ] **Forward-Compatible Schema Rule**: Every migration deployed in CD must follow the Expand-and-Contract pattern so the **currently running ($N-1$) application version** continues functioning normally while the migration executes and if the new ($N$) deployment rolls back.

### 3.4 Secret Management & Rotation
- [ ] **Zero Secrets in Git**: Enforce pre-commit and CI `gitleaks` scanning across commit history.
- [ ] **Dual-Key Zero-Downtime Secret Rotation**: Support an array of active verification keys (`JWT_SECRETS=["new_key", "old_key"]` or `WEBHOOK_SECRETS`) where new tokens are signed with `keys[0]` while verification accepts any key in the array during the rotation window.

### 3.5 Backup & Disaster Recovery (RPO / RTO Verification)
- [ ] **Define Explicit RPO & RTO Targets**:
  - **RPO (Recovery Point Objective)**: Maximum acceptable data loss window (e.g., `< 5 minutes` via continuous WAL archiving / Litestream replication for FinTech/Healthcare/E-commerce; `< 24 hours` for static content portals).
  - **RTO (Recovery Time Objective)**: Maximum acceptable time to restore service after catastrophic failure (e.g., `< 30 minutes`).
- [ ] **3-2-1 Encrypted Backup Architecture**:
  - Retain daily logical/physical snapshots plus continuous Write-Ahead Log (WAL) streaming encrypted with AES-256 (client-side encryption before upload) in an immutable/versioned off-site object storage bucket (e.g., Cloudflare R2 / Backblaze B2 with Object Lock).
- [ ] **Automated Restore Drill**: Schedule a recurring automated job (at least monthly) that spins up an ephemeral database container, restores the latest production backup, runs integrity queries (`pg_amcheck` / row count & checksum assertions), and alerts on failure.

---

## 4. Cloudflare Pages / Workers & Self-Hosted Linux VPS Hardening

### 4.1 Cloudflare Pages & Workers (`wrangler.toml`)
- [ ] **Compatibility Date Locking**: Pin a current `compatibility_date` and `compatibility_flags = ["nodejs_compat"]` in `wrangler.toml`.
- [ ] **Preview vs. Production Isolation**: Bind separate D1 databases, KV namespaces, R2 buckets, and Queues for `preview` and `production` environments so PR preview deployments never touch production data.
- [ ] **Secrets via `wrangler secret put`**: Never place secret tokens in `[vars]` inside `wrangler.toml`; use `wrangler secret put` or `.dev.vars` (gitignored) locally.

### 4.2 Self-Hosted Linux VPS Hardening (Ubuntu 24.04 / Debian 12)
- [ ] **SSH Hardening (`/etc/ssh/sshd_config`)**: Disable root login (`PermitRootLogin no`), disable password authentication (`PasswordAuthentication no`), enforce Ed25519 SSH keys only.
- [ ] **Host Firewall (`ufw` / `nftables`)**: Default deny all inbound traffic (`ufw default deny incoming`); allow only `80/tcp`, `443/tcp`, and SSH (or restrict SSH exclusively to a WireGuard / Tailscale mesh VPN interface!).
- [ ] **Docker & UFW Bypass Mitigation**: Be aware that publishing Docker ports (`-p 5432:5432`) manipulates `iptables` directly and **bypasses `ufw`**, exposing the database to the public internet! Always bind internal container ports to localhost (`127.0.0.1:5432:5432`) or an internal Docker network without host port mapping.
- [ ] **Unattended Security Upgrades**: Enable `unattended-upgrades` for automatic Linux kernel and security patching, paired with `fail2ban` or CrowdSec.
