---
name: role-github-admin
description: >-
  Act as a GitHub Repository Administrator & Automation Specialist to manage GitHub Projects (v2), labels, milestones, branch protections, issue creation/triage, PR workflows, and automated PR merging via GitHub CLI (gh) or REST API.
---

# GitHub Administrator & Automation Guide (`role-github-admin`)

## Mission
Manage the complete lifecycle of tasks, issues, branches, and pull requests on GitHub (or adaptable git hosting platforms such as GitLab or Gitea). Enable autonomous agent and human collaboration through structured issue tracking, PR automation, review enforcement, and clean merge strategies.

---

## 1. Authentication & Tooling Setup

To interact with GitHub autonomously, the environment requires one of two integration methods:

### Method A: GitHub CLI (`gh`)  -  Recommended
1. Install GitHub CLI:
   ```powershell
   winget install --id GitHub.cli
   ```
2. Authenticate the session:
   ```bash
   gh auth login
   ```
   - Select: `GitHub.com` -> `HTTPS` -> Authenticate via web browser or paste token.
3. Verify connection:
   ```bash
   gh auth status
   ```

### Method B: Fine-Grained Personal Access Token (PAT)
If running headless or in CI without interactive login:
1. Generate a token at `github.com/settings/tokens?type=beta` with:
   - **Repository permissions**: `Issues` (Read & Write), `Pull requests` (Read & Write), `Contents` (Read & Write), `Projects` (Read & Write).
2. Set environment variable:
   ```powershell
   $env:GH_TOKEN = "your_github_token_here"
   ```

---

## 2. GitHub CLI Workflows for Agents

### A. Searching & Listing Issues
```bash
# List open issues for the current repo
gh issue list --state open --limit 20

# View details of a specific issue
gh issue view 12
```

### B. Creating New Issues
```bash
# Create feature issue using template parameters
gh issue create \
  --title "feat(geo): implement location reverse geocoding" \
  --body "### Problem Statement\n...\n### Acceptance Criteria\n- [ ] 48px touch targets\n- [ ] Offline fallback" \
  --label "type: feat,area: ui"
```

### C. Branch & PR Lifecycle
1. **Create branch from issue**:
   ```bash
   git checkout -b feat/location-lookup
   ```
2. **Work and commit**:
   ```bash
   git commit -m "feat(geo): add reverse geocoding service

   - Add location detection action
   - Integrate geocoding API endpoint
   - Handle permissions denial gracefully

   Closes #12"
   ```
3. **Open Pull Request**:
   ```bash
   gh pr create \
     --title "feat(geo): add reverse geocoding service" \
     --body "Closes #12\n\n### Summary\n- Adds location lookup and error boundaries..." \
     --base main
   ```
4. **Review & Merge PR**:
   ```bash
   # Check CI status on PR
   gh pr checks

   # Merge using Squash & Merge (keeps main linear and clean)
   gh pr merge --squash --delete-branch
   ```

---

## 3. GitHub Projects (v2) Automation
- When an issue is created, link it to the project board:
  ```bash
  gh project item-create <project-number> --owner <org-or-user> --title "Task title"
  ```
- Update status field: `Backlog` -> `Ready for Dev` -> `In Progress` -> `In Review` -> `Done`.
