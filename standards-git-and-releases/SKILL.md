---
name: standards-git-and-releases
description: >-
  Enforce Staff-level Git hygiene including Conventional Commits 1.0.0, Semantic Versioning (SemVer 2.0.0), atomic commit granularity, linear trunk/feature-branch workflows, Pull Request traceability, and strict prohibition of unsolicited AI co-authorship attribution.
---

# Git Conventions, Branching & Release Standards (`standards-git-and-releases`)

## Mission
Maintain a clean, bisectable, self-documenting, and auditable Git history across the entire repository lifecycle. Enforce deterministic commit formatting, atomic changesets, linear squash-merge history, and strict compliance with the repository's AI authorship governance rules (`Rule F`).

---

## Activation & Cross-Linking Protocol

Whenever `standards-git-and-releases` is activated:
1. **Load Project Extensions (Rule C)**:
   - Check `<workspace>/.agents/extensions/standards-git-and-releases.md` for project-specific allowed commit scopes, release branch strategies, or automated changelog tools (`Changesets`, `semantic-release`, `Release Please`).
2. **Load Living Documentation (Rule D)**:
   - Cross-reference `<workspace>/docs/guide/` and `.agents/rules/git-conventions.md`.

> **Baseline Specifications (Rule G)**
> - **Commit Format**: Conventional Commits `v1.0.0` (`https://www.conventionalcommits.org/en/v1.0.0/`)
> - **Versioning Standard**: Semantic Versioning `v2.0.0` (`MAJOR.MINOR.PATCH`)
> - **Git Tooling**: Git `2.47+`, Commitlint `19.x` / Husky `9.x` / Lefthook `1.8+`

---

## 1. Conventional Commits Specification (`v1.0.0`)

Every commit message must strictly follow this structure:

```text
<type>(<scope>): <imperative summary in 50-65 chars or less>

<body: bulleted explanation of WHAT changed and WHY>

<footer: issue references (Closes #12, Refs #14) or BREAKING CHANGE notes>
```

### Mandatory Title Rules
1. **Allowed `<type>` Values**:
   - `feat`: Introduces a new user-facing or API feature (bumps `MINOR` in SemVer).
   - `fix`: Fixes a bug, regression, or data error (bumps `PATCH` in SemVer).
   - `perf`: Improves performance (Core Web Vitals, query speed, bundle size) without changing external behavior.
   - `refactor`: Restructures internal code without changing external behavior.
   - `docs`: Updates `/docs/`, ADRs, `README.md`, or inline code documentation.
   - `test`: Adds or hardens unit, integration, or Playwright E2E tests.
   - `ui` / `style`: Visual polish, design tokens, or formatting adjustments.
   - `chore` / `build` / `ci`: Scaffolding, dependencies, GitHub Actions, or project management config.
   - `security`: Hardens authentication, CSP, validation, or patches a vulnerability.
   - `revert`: Reverts a previous commit (`revert: feat(search): ...`).
2. **Functional `<scope>` Only (No Project Codenames — Rule A)**:
   - Scopes must describe the functional module (`auth`, `search`, `db`, `api`, `ui`, `seo`, `legal`, `e2e`, `ci`, `adr`, `pm`) — **never** a hardcoded project name or temporary codename.
3. **Imperative Mood, Lowercase, No Trailing Period**:
   - Write `"feat(auth): enforce rate limit on OTP verification"` (never `"Added rate limit."` or `"Fixes bug in OTP"`).
   - Keep the subject line `<= 65` characters (`72` hard ceiling).

### Informative Commit Body & Issue Traceability
For any non-trivial commit, include a blank line followed by:
- Concise `- ` bullet points explaining *what* changed and *why* the architectural approach was chosen.
- An explicit issue reference footer (`Closes #14` when completing an issue, or `Refs #14` for intermediate commits).

```text
feat(search): add PostGIS bounding-box query and HTML directory fallback

- Implement spatial GIST index lookup for map viewport queries
- Add server-rendered hierarchical region/city links for search crawler parity
- Validate lat/lng coordinate bounds with Zod schema at controller boundary

Closes #24
```

---

## 2. Non-Negotiable AI Authorship & Attribution Policy (`Rule F`)

> [!IMPORTANT]
> **AI coding agents must NEVER append themselves as commit authors or co-authors (`Co-authored-by:`) unless explicitly instructed by the user.**

- **Strictly Prohibited by Default**:
  - Do **NOT** add `Co-authored-by: Claude...`, `Co-authored-by: Gemini...`, `Co-authored-by: Copilot...`, or `Generated with [Agent]` trailers to commit messages or Pull Request bodies.
  - Do **NOT** override `git config user.name` or `git config user.email` with bot identities.
- **One-Time Project Inception Question**:
  - Only when initializing a brand-new repository from scratch (`init-project`) may the agent ask the user once if they want AI attribution enabled. If the user declines or does not explicitly request it, default strictly to **zero AI attribution**.

---

## 3. Branching Strategy, Atomic Commits & Linear History

1. **Short-Lived Feature Branches**:
   - Branch directly from an up-to-date `main`:
     - `feat/<issue-id>-<short-slug>` (e.g., `feat/18-accessible-filter-drawer`)
     - `fix/<issue-id>-<short-slug>` (e.g., `fix/22-dst-timezone-offset`)
     - `docs/<issue-id>-<short-slug>` (e.g., `docs/05-adr-database-selection`)
2. **Atomic Commits**:
   - Each commit should represent a single logical, compiling, test-passing unit of work. Never mix a massive formatting/linter sweep with a critical security bugfix in the same commit.
3. **Pre-Commit Verification**:
   - Never commit code with failing typechecks (`tsc --noEmit`), failing unit tests, leftover `console.log` debugging, or accidentally staged `.env` secrets.
4. **Squash & Merge to `main`**:
   - Merge Pull Requests into `main` via **Squash and Merge** (or clean Rebase) so every commit on `main` corresponds 1:1 with a verified, review-approved vertical deliverable and passes `git bisect`.

---

## 4. Semantic Versioning (`SemVer 2.0.0`) & Release Hygiene

- **`MAJOR` (`vX.0.0`)**: Incompatible API contract changes or major platform overhauls (`BREAKING CHANGE:` in commit footer or `feat!:` / `fix!:`).
- **`MINOR` (`v0.X.0` / `v1.X.0`)**: Backward-compatible new features (`feat:`).
- **`PATCH` (`v1.0.X`)**: Backward-compatible bug fixes, security patches, and performance tuning (`fix:`, `security:`, `perf:`).

---

## Mandatory Git & Release Checklist

- [ ] **Conventional Commit Format**: Title matches `<type>(<scope>): <imperative summary>` (`<= 65` chars, lowercase, no period).
- [ ] **No Project Codenames**: Scope reflects a functional module (`api`, `ui`, `auth`), never the project name.
- [ ] **Zero AI Co-Authorship (`Rule F`)**: Confirmed zero `Co-authored-by:` bot lines or agent branding in the commit message.
- [ ] **Staged Diff Audit**: Ran `git status` and `git diff --staged` before committing to ensure zero secrets (`.env`), build artifacts (`dist/`, `.next/`), or scratch files are staged.
- [ ] **Issue Linked**: Footer includes `Closes #<id>` or `Refs #<id>`.

---

## Continuous Evolution & Self-Improvement Mandate
- **Baseline Floor, Not a Ceiling**: Enforce these standards mechanically via `commitlint` and git hooks (`lefthook` / `husky`) during Phase 6 scaffolding so human and AI contributors receive instant feedback.
- **Knowledge Capture**: Record project-specific commit scopes and release pipelines in `<workspace>/.agents/extensions/standards-git-and-releases.md`.
