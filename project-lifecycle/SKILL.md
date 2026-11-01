---
name: project-lifecycle
description: >-
  Master project lifecycle and phase progression guide enforcing strict sequencing: discovery, architecture, governance, living docs, visual Penpot design, core scaffolding, design system code implementation, and feature delivery loops.
---

# Project Lifecycle & Feature Delivery Workflow

```mermaid
flowchart TD
    S0["0. Product Concept & Scope Discovery<br/>(role-product-owner, role-business-analyst)"] --> S1["1. Architecture & FOSS Stack Selection<br/>(role-software-architect, tooling-free-and-open-source)"]
    S1 --> S2["2. Repository Governance & PM Setup<br/>(role-github-admin, standards-issue-and-project-management)"]
    S2 --> S3["3. Archivist Baseline & Living Docs (/docs)<br/>(role-archivist)"]
    S3 --> S4["4. Visual UI/UX Design & Wireframing (Penpot)<br/>(role-ui-ux-designer, tooling-free-and-open-source)"]
    S4 --> S5["5. Core Technical Scaffolding & Data Model<br/>(project-agent-scaffolder, role-backend-developer)"]
    S5 --> S6["6. Design System Implementation (Code Tokens & Atomic UI)<br/>(design-engineering, role-frontend-developer)"]
    S6 --> S7["7. Feature Implementation Loop<br/>(TDD, UI Craft, QA, PR Review, Release)"]
```

## Mandatory Phase Order

1. **Phase 0: Discovery & Scope**: Elicit problem statement, target users, and in/out of scope boundaries.
2. **Phase 1: Architecture & FOSS Stack**: Select tech stack adhering to zero-cost/open-source policies.
3. **Phase 2: Repository Governance & PM Setup**: Setup GitHub Project (v2), issue templates, milestones, and labels.
4. **Phase 3: Archivist Baseline (`/docs`)**: Capture foundational Architecture Decision Records (ADRs) and framework-agnostic domain models.
5. **Phase 4: Visual UI/UX Design & Wireframing in Penpot (Zero Code)**:
   - Establish visual design tokens (typography, color palettes, spacing, elevation) in Penpot.
   - Design screen wireframes and component layouts for visual inspection and human sign-off *before writing any UI code*.
6. **Phase 5: Technical Scaffolding & Data Model**:
   - Initialize application skeleton (Next.js/React, TypeScript strict mode, build toolchain).
   - Configure database collections/schemas (Payload CMS) and CI pipeline.
7. **Phase 6: Design System Implementation in Code**:
   - Translate verified Penpot design tokens into code (Tailwind configuration).
   - Build reusable atomic UI component library (buttons, inputs, cards, badges) on top of the established application skeleton.
8. **Phase 7: Feature Implementation Loop**:
   - Assemble full feature screens and workflows using the verified data model and atomic UI components.
