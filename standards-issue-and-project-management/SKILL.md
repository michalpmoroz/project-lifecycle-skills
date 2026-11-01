---
name: standards-issue-and-project-management
description: >-
  Establish structured GitHub Issue templates, label taxonomies, milestone management, and GitHub Projects (v2) Kanban workflows for end-to-end task tracking.
---

# GitHub Issues & Project Management Standards

## Mission
Maintain crystal-clear project transparency and structured task lifecycles using GitHub Issues, Labels, Milestones, and GitHub Projects (v2).

## Issue Taxonomy & Templates

### 1. Issue Types:
- **Epic**: High-level capability spanning multiple features (`[Epic] User Authentication & Onboarding`).
- **Feature**: User-facing capability with user story and acceptance criteria.
- **Bug**: Defect report with reproduction steps and device details.
- **Task / Chore**: Technical maintenance, refactoring, CI/CD adjustments.

### 2. Standard GitHub Labels:
- **Type**: `type: feat`, `type: bug`, `type: chore`, `type: docs`, `type: epic`
- **Priority**: `priority: critical`, `priority: high`, `priority: medium`, `priority: low`
- **Area**: `area: mobile`, `area: ui`, `area: api`, `area: pwa`, `area: devops`
- **Status**: `status: ready`, `status: blocked`, `status: in-progress`

### 3. Issue Template Specifications

#### Feature Request (`.github/ISSUE_TEMPLATE/feature_request.yml`):
- Problem statement / user story.
- Proposed solution & UI expectations.
- Acceptance criteria (checkboxes).
- Mobile considerations (touch targets, offline behavior).

#### Bug Report (`.github/ISSUE_TEMPLATE/bug_report.yml`):
- Steps to reproduce.
- Expected behavior vs Actual behavior.
- Environment: OS (iOS/Android/Windows/macOS), browser/version, device model.
- Error logs or screenshots.

### 4. GitHub Projects (v2) Board Setup
Columns:
1. **Backlog / Triage**: Unrefined or newly submitted issues.
2. **Ready for Dev**: Scoped, labeled, and prioritized tasks.
3. **In Progress**: Assigned to active developer/branch.
4. **In Review**: Pull request opened, undergoing review and CI.
5. **Done**: Merged and verified in deployment.

