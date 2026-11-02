---
name: standards-code-style
description: >-
  Enforce universal, tool-agnostic baseline code style, text formatting, cross-language naming conventions, module import ordering, complexity budgets, and static analysis rules across TypeScript/JavaScript, HTML/CSS/SCSS/Tailwind, SQL, backend services, and automated test suites.
---

# Baseline Code Style, Formatting & Static Analysis Standards (`standards-code-style`)

## Mission
Establish a deterministic, high-signal, tool-agnostic baseline for code formatting, naming conventions, structural organization, complexity budgets, and static analysis across all project lifecycles. Eliminate bikeshedding in Pull Requests, prevent subtle runtime defects (floating promises, implicit type coercion, CSS specificity wars, SQL naming drift), and ensure that code authored by humans and AI agents reads as a single, cohesive Staff-engineered system.

> [!IMPORTANT]
> **Tool-Agnostic Canonical Principles (No Hardcoded Config Files in Base Skills)**
> This skill codifies the **canonical engineering rules and invariants** in Markdown rather than shipping static `.prettierrc` or `eslint.config.js` files inside the skills repository. Target projects consume these rules in two ways:
> 1. **Direct Code Authoring & Review**: Agents and developers apply these rules immediately when writing or reviewing code in any repository.
> 2. **Phase 6 Scaffolding Translation**: During project initialization (`init-project` Phase 6), agents translate these baseline rules into the target repository's chosen toolchain configurations (`.editorconfig`, Biome `biome.json`, ESLint 9 `eslint.config.mjs` + Prettier, Stylelint, Ruff `pyproject.toml`, `golangci-lint`, or `sqlfluff`).

---

## Activation & Cross-Linking Protocol

Whenever `standards-code-style` is activated:
1. **Load Project Extensions (Rule C)**:
   - Check `<workspace>/.agents/extensions/standards-code-style.md` for project-specific overrides (e.g., 4-space vs. 2-space team conventions, legacy naming exceptions, custom workspace path aliases, or domain-specific banned APIs).
2. **Load Living Documentation (Rule D)**:
   - Check `<workspace>/docs/adr/` and `<workspace>/docs/domain/glossary.md` (and documents tagged with `standards-code-style` in `skills:` YAML frontmatter) so identifier naming aligns 100% with the project's Ubiquitous Language.
3. **Load Selective Reference Guides (Rule B & Rule G)**:
   - Load the specific reference file(s) matching the language or layer being authored, scaffolded, or reviewed:

| Reference File | Baseline Specification & Tool Versions | When to Load |
| :--- | :--- | :--- |
| [`references/formatting-naming-and-structure.md`](file:///d:/project-lifecycle-skills/standards-code-style/references/formatting-naming-and-structure.md) | UTF-8, ECMAScript `2025`, TypeScript `5.7+`, Python `PEP 8`, Go `1.23+`, Rust `2024` | Always load when naming files/variables/types, organizing imports, setting complexity budgets, or generating `.editorconfig` / formatter configs. |
| [`references/javascript-typescript-lint-rules.md`](file:///d:/project-lifecycle-skills/standards-code-style/references/javascript-typescript-lint-rules.md) | ECMAScript `2025`, TypeScript `5.7+` (`strict`), ESLint `9.x` / `typescript-eslint` `8.x` / Biome `1.9+` | Writing, reviewing, or linting any JavaScript or TypeScript code (frontend, backend Node/Bun/Deno, shared domain packages). |
| [`references/html-css-scss-style-rules.md`](file:///d:/project-lifecycle-skills/standards-code-style/references/html-css-scss-style-rules.md) | HTML Living Standard (`2026`), CSS Snapshot `2025/2026`, Dart Sass `1.80+`, Tailwind CSS `4.0`, WCAG `2.2 AA` | Authoring or reviewing HTML templates, JSX/TSX/Vue/Svelte markup, CSS/SCSS/LESS stylesheets, or Tailwind utility classes. |
| [`references/sql-backend-and-test-style-rules.md`](file:///d:/project-lifecycle-skills/standards-code-style/references/sql-backend-and-test-style-rules.md) | SQL:`2023`, PostgreSQL `17`, Vitest `2.x` / Jest `29+`, Playwright `1.49+` | Designing database schemas, writing SQL queries/migrations, structuring backend services, or authoring unit/integration/E2E test suites. |

---

## Core Architectural Style Principles

### 1. Determinism Over Personal Preference
- Formatting (indentation, quotes, semicolons, trailing commas, import sorting, CSS property ordering) must be **100% deterministic and mechanically enforced** on save and in CI (`format:check`).
- Code reviews must never waste human energy debating whitespace; if a formatting rule matters, encode it in the project's formatter and `.agents/extensions/standards-code-style.md`.

### 2. Semantic Clarity & Ubiquitous Language
- Names must reveal intent, domain role, and physical/financial units (`settlementDelayMs`, `unitPriceMinorUnits`, `isEligibleVoter`, `hasActivePrescription`).
- Never use cryptic abbreviations (`val`, `tmp`, `mgr`, `proc`, `arr`, `obj`, `cb`) or single-letter variables outside trivial 1–2 line lambda indices.

### 3. Parse, Don't Validate — And Fail at Compile Time
- Static analysis rules exist to promote runtime bugs into compile-time errors: ban `any`, ban unhandled promises, ban implicit falsy coercion on numbers/strings, and require exhaustive `switch` checks (`satisfies never`).

### 4. Flat Control Flow & Bounded Complexity
- Deeply nested `if / else` pyramids hide edge cases. Enforce guard clauses (early returns), extract cohesive helper functions, and respect strict cyclomatic (`<= 10`) and cognitive (`<= 15`) complexity ceilings.

---

## Mandatory Baseline Code Style Checklist

Before committing code, approving a Pull Request, or finalizing Phase 6 repository scaffolding, verify:

- [ ] **Text & Whitespace Hygiene**: UTF-8 (no BOM), `LF` (`\n`) line endings, `2 spaces` indentation for web/TS/JS/CSS/SQL/JSON/YAML (`4 spaces` for Python/Rust, `tabs` for Go), `100`-character target line width (`120` hard ceiling), single trailing newline, zero trailing whitespace.
- [ ] **Naming Consistency**: Files in `kebab-case` with architectural suffixes (`.service.ts`, `.component.ts`, `.spec.ts`); booleans prefixed with positive predicates (`is*`, `has*`, `can*`, `should*`); functions prefixed with imperative verbs; zero `I` prefixes on interfaces or `T` suffixes on types; physical/financial units suffixed (`Ms`, `Bytes`, `MinorUnits`/`Cents`).
- [ ] **Import & Module Discipline**: 5-tier ordered imports (Built-in -> External -> Internal Aliases -> Relative -> Type-only `import type`), alphabetized within groups, zero circular dependencies, zero deep relative parent climbs (`../../../`).
- [ ] **Complexity Budgets**: Cyclomatic complexity `<= 10`, cognitive complexity `<= 15`, max nesting depth `<= 3`, max function length `<= 60` lines, max file length `<= 350` lines, max positional parameters `<= 3`.
- [ ] **JS/TS Static Analysis Gates**: Zero `any`, zero `@ts-ignore`, zero non-null `!` assertions, zero floating/misused promises, `return await` inside `try/catch`, `??` over `||` for defaults, immutable array operations (`.toSorted()`), zero `console.log` in production paths.
- [ ] **HTML/CSS/Tailwind Craft**: Explicit `type="button"` on non-submit buttons, zero `#id` selectors or `!important` in component styles, logical CSS properties (`margin-inline`, `padding-block`), semantic design tokens (`var(--color-*)`), `rem` typography/spacing, `@use`/`@forward` in SCSS, canonical Tailwind class sorting with zero dynamic string interpolation (`text-${color}-500`).
- [ ] **SQL & Test Suite Discipline**: `UPPERCASE` SQL keywords, `lower_snake_case` identifiers, plural table names, singular column names, explicit column lists (no `SELECT *`), deterministic constraint prefixes (`pk_`, `fk_`, `uq_`, `chk_`, `idx_`), and strict **Arrange–Act–Assert (AAA)** test structure with zero conditional logic inside tests.

---

## Continuous Evolution & Self-Improvement Mandate
- **Baseline Floor, Not a Ceiling**: These rules establish the minimum engineering bar across languages. When a project adopts newer language features or discovers recurring defect patterns in code review, immediately codify new lint rules in the project's toolchain and record deltas in `<workspace>/.agents/extensions/standards-code-style.md`.
- **Upstream Universal Learnings**: When a newly discovered static analysis rule or style invariant proves universally valuable across domains, upstream it into the reference files of this skill with updated baseline version metadata (Rule G).
