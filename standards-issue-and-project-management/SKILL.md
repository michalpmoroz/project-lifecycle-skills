---
name: standards-issue-and-project-management
description: >-
  Establish structured GitHub Issue Forms (.github/ISSUE_TEMPLATE/*.yml), deterministic label taxonomies, milestone planning, Definition of Ready/Done gates, and GitHub Projects (v2) Kanban workflows for end-to-end engineering traceability.
---

# GitHub Issues & Project Management Standards (`standards-issue-and-project-management`)

## Mission
Ensure 100% traceability from initial product idea to merged Pull Request using structured GitHub Issue Forms, deterministic label taxonomies, milestone gates, and GitHub Projects (v2) Kanban automation. Guarantee that any human developer or AI agent inspecting the repository backlog can immediately understand what needs to be built, why it matters, what blocks it, and how to verify it.

---

## Activation & Cross-Linking Protocol

Whenever `standards-issue-and-project-management` is activated:
1. **Load Project Extensions (Rule C)**:
   - Check `<workspace>/.agents/extensions/standards-issue-and-project-management.md` for project-specific area labels (`area: billing`, `area: telehealth`, `area: geospatial`), custom SLA priorities, or sprint/milestone naming rules.
2. **Load Related Governance Skills (Rule D)**:
   - Coordinate directly with `role-project-manager`, `role-product-owner`, `role-github-admin`, and `standards-omni-role-review`.

> **Baseline Tooling & Schema Standards (Rule G)**
> - **Issue Templates**: GitHub Issue Forms YAML Schema (`.github/ISSUE_TEMPLATE/*.yml`)
> - **Board Management**: GitHub Projects (v2) (`Status`, `Priority`, `Size`, `Milestone`, `Iteration` fields)
> - **Review Gate**: `standards-omni-role-review` 14-Role Verification Protocol

---

## 1. Issue Hierarchy & Taxonomy

Every unit of work in the repository must be represented by one of five structured issue types:

1. **Epic (`[Epic] <Capability Name>`)**:
   - High-level product initiative spanning multiple vertical user stories (e.g., `[Epic] Multi-Provider Appointment Booking & Calendar Sync`). Contains a tasklist tracking child issues (`- [ ] #12`).
2. **Feature / Story (`feat(<scope>): <capability>`)**:
   - Vertically sliced, user-facing capability (`XS`, `S`, or `M` size) with Gherkin acceptance criteria (`Given/When/Then`) and all 8 UI states defined.
3. **Bug Report (`bug(<scope>): <observed failure>`)**:
   - Confirmed defect with deterministic reproduction steps, viewport/device environment, expected vs. actual behavior, and a mandate to write a failing test before fixing.
4. **Technical Task / Chore (`chore(<scope>): <task>`)**:
   - Scaffolding, CI/CD pipeline setup, database migration, dependency upgrade, or performance/security hardening.
5. **Architecture Decision Proposal (`adr: <decision topic>`)**:
   - Tracks a technical spike (`role-tech-researcher`) or architectural trade-off culminating in a committed `/docs/adr/XXXX-*.md` file.

---

## 2. Deterministic Label Taxonomy

Every repository must configure four orthogonal label namespaces:

| Namespace | Standard Labels | Purpose |
| :--- | :--- | :--- |
| **`type:*`** | `type: epic`, `type: feat`, `type: bug`, `type: chore`, `type: docs`, `type: security`, `type: adr` | Classifies the nature of the deliverable. |
| **`priority:*`** | `priority: p0-critical`, `priority: p1-high`, `priority: p2-medium`, `priority: p3-low` | Orders execution along the critical path. |
| **`size:*`** | `size: xs`, `size: s`, `size: m`, `size: l-needs-split` | Enforces the 1–2 session sizing limit (`l-needs-split` cannot be started without decomposition). |
| **`area:*`** | `area: ui`, `area: api`, `area: db`, `area: auth`, `area: seo-geo`, `area: legal`, `area: qa`, `area: ci` | Routes tasks to the appropriate specialist role skills. |
| **`status:*`** | `status: triage`, `status: ready`, `status: in-progress`, `status: blocked`, `status: in-review` | Mirrors Kanban board state for CLI queries (`gh issue list --label "status: ready"`). |

---

## 3. GitHub Issue Form Templates (`.github/ISSUE_TEMPLATE/`)

Use structured YAML Issue Forms (rather than freeform Markdown templates) so required fields cannot be skipped:

### Feature Request Form (`.github/ISSUE_TEMPLATE/01-feature.yml`)
```yaml
name: "Feature / User Story"
description: "Propose a vertically sliced user feature with Gherkin acceptance criteria"
title: "feat(scope): "
labels: ["type: feat", "status: triage"]
body:
  - type: textarea
    id: story
    attributes:
      label: "User Story & Value (role-product-owner)"
      description: "As a <persona>, I want <capability>, so that <measurable outcome>."
      placeholder: "As a mobile user searching for..."
    validations:
      required: true
  - type: textarea
    id: gherkin
    attributes:
      label: "Acceptance Criteria — Gherkin (Happy Path, Empty/Boundary, Error)"
      description: "Define deterministic Given / When / Then scenarios."
      value: |
        Scenario: Primary happy path
          Given ...
          When ...
          Then ...

        Scenario: Empty or boundary state
          Given ...
          When ...
          Then ...

        Scenario: Error, offline, or permission denied state
          Given ...
          When ...
          Then ...
    validations:
      required: true
  - type: checkboxes
    id: ui_states
    attributes:
      label: "UI, Mobile & Compliance Requirements (standards-omni-role-review)"
      options:
        - label: "Mobile-first (390x844) layout & >= 48x48px touch targets verified"
        - label: "All 8 UI states (Loading, Empty, Partial, Ideal, Overflow, Error, Offline, Permission) specified"
        - label: "GDPR privacy, source citation & SEO/JSON-LD impact evaluated"
        - label: "Playwright E2E + axe-core WCAG 2.2 AA test cases planned"
  - type: textarea
    id: dependencies
    attributes:
      label: "Dependencies & Out of Scope"
      placeholder: "- Depends on #\n- Out of scope for this issue:"
```

### Bug Report Form (`.github/ISSUE_TEMPLATE/02-bug.yml`)
```yaml
name: "Bug / Regression Report"
description: "Report a reproducible defect with viewport and environment details"
title: "bug(scope): "
labels: ["type: bug", "status: triage"]
body:
  - type: textarea
    id: repro
    attributes:
      label: "Deterministic Steps to Reproduce"
      value: |
        1. Navigate to '...' on viewport (e.g., 390x844 mobile / 1440x900 desktop)
        2. Tap / Click on '...'
        3. Observe error
    validations:
      required: true
  - type: textarea
    id: expected_actual
    attributes:
      label: "Expected vs. Actual Behavior"
    validations:
      required: true
  - type: input
    id: environment
    attributes:
      label: "Environment (OS, Browser, Viewport, Commit SHA)"
      placeholder: "iOS 18 Safari (390x844) / Commit a1b2c3d"
    validations:
      required: true
```

---

## 4. GitHub Projects (v2) Kanban Workflow

Every repository board must enforce five sequential workflow columns:
1. **`Inbox / Triage`**: Newly created issues awaiting `standards-omni-role-review` refinement and sizing.
2. **`Ready for Dev`**: Satisfies Definition of Ready (DoR), sized `XS`/`S`/`M`, and has zero unresolved blocking dependencies.
3. **`In Progress`**: Active feature branch checked out (`feat/<issue-id>-<slug>`). Limit Work-in-Progress (WIP) to 1 active issue per agent/developer.
4. **`In Review`**: Pull Request opened (`Closes #<issue-id>`), automated CI running, undergoing `role-senior-code-reviewer` and `standards-omni-role-review` audit.
5. **`Done`**: PR squash-merged to `main`, CI green, `/docs/` updated, and issue automatically closed.

---

## Mandatory Issue & PM Governance Checklist

- [ ] **YAML Issue Forms Installed**: `.github/ISSUE_TEMPLATE/` contains structured YAML forms for Features, Bugs, and Tasks/ADRs, plus `.github/pull_request_template.md`.
- [ ] **Labels & Milestones Provisioned**: Standard `type:*`, `priority:*`, `size:*`, `area:*`, and `status:*` labels and lifecycle Milestones exist in the repository.
- [ ] **No Unrefined Coding**: Zero feature code is written without a corresponding `Ready for Dev` issue containing Gherkin criteria and 360-degree review sign-off.
- [ ] **WIP Discipline**: Only one task per agent is in `In Progress` at a time; completed PRs close their linked issue via `Closes #<id>`.

---

## Continuous Evolution & Self-Improvement Mandate
- **Baseline Floor, Not a Ceiling**: Treat these templates and board workflows as the foundational governance floor. Customize domain-specific checklists inside `<workspace>/.agents/extensions/standards-issue-and-project-management.md`.
