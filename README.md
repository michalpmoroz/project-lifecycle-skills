# 🧭 Project Lifecycle Skills

[![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Standards: Project--Agnostic](https://img.shields.io/badge/Standards-Project--Agnostic-success.svg)](standards-skills-governance/SKILL.md)
[![AI Attribution Policy](https://img.shields.io/badge/AI%20Attribution-Zero%20Co--Author-red.svg)](AGENTS.md)

**Project Lifecycle Skills** is an end-to-end, project-agnostic suite of operational skills, specialized engineering roles, and quality standards for AI coding agents and human engineering teams.

Designed to eliminate "AI coding hallucinations" and disorganized development, this skillset enforces phased project lifecycles, rigorous omni-role reviews, architectural governance, mobile-first design engineering, living documentation, and automated GitHub workflows.

---

## 🚀 Capabilities Overview

The skillset spans **30 specialized modules** organized into four pillars:

### 1. Orchestration & Lifecycle Workflows
- **`init-project`**: Master entry point to scaffold a project from scratch with phased discovery, stack selection, web essentials audit, and archivist setup.
- **`continue-project`**: Master resume point to audit repo state, inspect backlog items, diagnose blockers, and propose prioritized next tasks.
- **`project-lifecycle`**: Strict 8-phase sequential progression engine (Discovery ➔ Architecture ➔ Governance ➔ Docs ➔ Penpot Design ➔ Scaffolding ➔ Design System ➔ Features).
- **`project-agent-scaffolder`**: Scaffolder for directory-scoped agent instructions (`AGENTS.md`, `GEMINI.md`, and `.agents/rules/`).

### 2. Specialized Roles (Omni-Role Suite)
- **Strategy & Governance**: `role-product-owner`, `role-business-analyst`, `role-project-manager`, `role-github-admin`.
- **Architecture & System Design**: `role-software-architect`, `role-tech-researcher`, `role-archivist`.
- **Experience & Design Engineering**: `role-ui-ux-designer`, `design-engineering`, `role-copywriter`.
- **Implementation & Platforms**: `role-frontend-developer`, `role-backend-developer`, `role-mobile-developer`.
- **Verification & Review**: `role-qa-engineer`, `role-senior-code-reviewer`.
- **Security & Regulatory**: `role-security-consultant` (OWASP/threat modeling), `role-legal-counsel` (GDPR/privacy/liability).
- **Operations & Discovery**: `role-devops-engineer` (CI/CD), `role-performance-consultant` (Core Web Vitals), `role-seo-geo-specialist`.

### 3. Standards & Protocols
- **`standards-omni-role-review`**: Mandatory 360-degree review protocol evaluating PRs and specs across all 14 perspectives.
- **`standards-skills-governance`**: Core governance laws (Project-Agnostic, Tech Helper files, Extension Protocol, Docs Cross-Linking, English-Only, Zero AI Co-Authorship).
- **`standards-web-essentials`**: Pre-backlog audit checklist covering Sitemaps, Robots, Open Graph, Error Pages, Legal policies, and Security.txt.
- **`standards-git-and-releases`**: Conventional Commits, SemVer, branch protections, and PR release flow.
- **`standards-issue-and-project-management`**: Issue templates, label taxonomies, milestones, and GitHub Projects v2 automation.

### 4. Tooling Philosophy
- **`tooling-free-and-open-source`**: Enforces Free & Open-Source First policy, pairing proprietary standards with free/open-source alternatives (Penpot, PocketBase/Supabase, Playwright, Cloudflare Pages/Workers, Docker).

---

## 🏗️ Recommended Project Structure

To maximize effectiveness when consuming Project Lifecycle Skills, configure your application repository with the following layout:

```text
my-awesome-project/
├── .agents/
│   ├── extensions/                    # 💡 PROJECT EXTENSIONS (Rule C)
│   │   ├── role-frontend-developer.md # Project-specific framework rules & styling
│   │   ├── role-backend-developer.md  # Custom DB schemas, ORM rules, auth logic
│   │   ├── role-legal-counsel.md      # Jurisdiction details, legal disclaimers
│   │   ├── role-seo-geo-specialist.md # Sitemap partitions, local GEO hierarchy
│   │   └── standards-git-and-releases.md # Project commit scopes
│   └── rules/                         # Scoped rules referenced by agents
│       ├── git-conventions.md
│       └── mobile-standards.md
│
├── .github/
│   ├── ISSUE_TEMPLATE/                # Managed via standards-issue-and-project-management
│   └── workflows/                     # CI/CD pipelines managed via role-devops-engineer
│
├── docs/                              # 📚 LIVING DOCUMENTATION (Hard Drive Persistence)
│   ├── adr/                           # Architecture Decision Records (role-software-architect)
│   ├── architecture/                  # System blueprints, data models, contracts
│   ├── audits/                        # Web essentials, security, legal, SEO audits
│   ├── research/                      # Spikes, competitor benchmarks, tech trade-offs
│   └── design/                        # Penpot token exports, component hierarchies
│
├── src/                               # Application source code
│   ├── app/ (frontend)
│   │   └── AGENTS.md                  # Frontend-scoped agent directives
│   ├── server/ (backend)
│   │   └── AGENTS.md                  # Backend-scoped agent directives
│   └── shared/
│       └── AGENTS.md                  # Shared types / contracts directives
│
├── tests/
│   └── AGENTS.md                      # QA & E2E testing directives
│
├── AGENTS.md                          # Root agent instructions linking to skills
└── README.md
```

---

## 🛠️ How to Use & Install

### Option A: Global Configuration (User-wide for Antigravity / Agentic IDEs)
Copy the skills into your global agent configuration directory:
```powershell
# Windows
Copy-Item -Path ".\*" -Destination "$HOME\.gemini\config\skills\" -Recurse -Force

# Linux / macOS
cp -r ./* ~/.gemini/config/skills/
```

### Option B: Project-Level Usage
Embed or submodule the skills within your project under `.agents/skills/` or `.github/skills/`, and configure your AI assistant's root instructions (`AGENTS.md` or `.cursorrules`) to reference them.

---

## 🧩 Extending Skills (Without Modifying Base Skills)

Base skills are strictly **project-agnostic** and must never be edited for individual project needs.

To extend or customize rules for your project:
1. Create a file in `<project-root>/.agents/extensions/<skill-name>.md`.
2. Follow the extension template:
   ```markdown
   # role-frontend-developer - Project Extension

   ## Project Context
   Building a mobile-first real-time dashboard using Angular Signals and Tailwind CSS.

   ## Overrides
   - Use Tailwind v4 `@theme` tokens instead of CSS modules.
   - Route slugs use localized paths for public marketing pages.

   ## Additions
   - Charting library: Apache ECharts.
   - State management: NgRx ComponentStore for complex tabular states.
   ```
3. Whenever an agent activates `role-frontend-developer`, it automatically merges your project extension over the base skill instructions.

---

## 📖 Documentation Cross-Linking Protocol

All research, architecture, and design documents in `/docs/` should declare which skills they pertain to using YAML frontmatter:

```markdown
---
skills: [role-archivist, role-backend-developer, role-legal-counsel]
---

# User Data Deletion & Privacy Policy Compliance
...
```

When an agent assumes a role, it queries `/docs/` for relevant documents containing its skill name in the frontmatter, ensuring context is instantly retrieved.

---

## 🤖 Directives for AI Agents

All AI coding assistants (Antigravity, Cursor, Claude Code, GitHub Copilot) operating within this repository or consuming these skills **must follow [AGENTS.md](AGENTS.md)**:
- **Zero AI Co-Authorship**: Never add `Co-authored-by:` or AI signatures to commits unless explicitly requested.
- **Project-Agnosticism**: Never hardcode single-project business logic or URLs into base skills.
- **English-Only**: All skill content must be in English.
- **Hard Drive Persistence**: Persist research, plans, and ADRs directly to `<workspace>/docs/`.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).
