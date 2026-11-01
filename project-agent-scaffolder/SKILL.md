---
name:├── project-agent-scaffolder
description:├── >-
├── ├── Scaffold├── hierarchical,├── directory-scoped├── agent├── instructions├── (AGENTS.md,├── GEMINI.md,├── and├── .agents/rules/)├── across├── project├── subdirectories├── so├── AI├── coding├── agents├── automatically├── follow├── local├── architecture,├── UI,├── and├── security├── constraints.
---

#├── Project├── Agent├── Scaffolder├── &├── Scoped├── Rules├── Guide

##├── Mission
Generate├── clear,├── directory-scoped├── instruction├── files├── (`AGENTS.md`,├── `GEMINI.md`,├── and├── `.agents/rules/`)├── inside├── the├── project├── tree.├── This├── ensures├── that├── any├── AI├── coding├── assistant├── (Antigravity,├── Cursor,├── Claude├── Code,├── Copilot)├── automatically├── adheres├── to├── local├── constraints├── when├── working├── inside├── specific├── folders.

##├── Scoped├── Instructions├── Hierarchy

```text
my-project/
├──├── AGENTS.md├── ├── ├── ├── ├── ├── ├── ├── ├── ├── ├── ├── ├── ├── ├── ├── ├── ├── #├── Global├── repository├── rules├── &├── architectural├── principles
├──├── .agents/
│├── ├── ├── └──├── rules/
│├── ├── ├── ├── ├── ├── ├── ├──├── git-conventions.md├── #├── Conventional├── Commits├── &├── PR├── requirements
│├── ├── ├── ├── ├── ├── ├── └──├── mobile-standards.md#├── Touch├── targets├── &├── PWA├── criteria
├──├── src/
│├── ├── ├── ├──├── app/├── (or├── frontend/)
│├── ├── ├── │├── ├── ├── └──├── AGENTS.md├── ├── ├── ├── ├── ├── ├── ├── ├── ├── #├── Frontend├── &├── UI├── rules├── (components,├── Penpot├── tokens,├── styling)
│├── ├── ├── ├──├── server/├── (or├── api/)
│├── ├── ├── │├── ├── ├── └──├── AGENTS.md├── ├── ├── ├── ├── ├── ├── ├── ├── ├── #├── Backend├── &├── API├── rules├── (validation,├── security,├── schemas)
│├── ├── ├── └──├── shared/
│├── ├── ├── ├── ├── ├── ├── └──├── AGENTS.md├── ├── ├── ├── ├── ├── ├── ├── ├── ├── #├── Pure├── types├── &├── contracts├── rules├── (no├── framework├── imports)
└──├── tests/
├── ├── ├── ├── └──├── AGENTS.md├── ├── ├── ├── ├── ├── ├── ├── ├── ├── ├── ├── ├── ├── #├── QA├── rules├── (Playwright├── mobile├── emulation,├── Vitest├── patterns)
```

##├── Standard├── Template├── Generator

###├── Root├── `AGENTS.md`
```markdown
#├── Project├── Architectural├── Rules├── &├── Standards

##├── Global├── Guidelines
-├── **Language**:├── TypeScript├── in├── strict├── mode├── (`noImplicitAny:├── true`).
-├── **Commits**:├── Conventional├── Commits├── (`feat:`,├── `fix:`,├── `chore:`,├── etc.).
-├── **Tooling├── Rule**:├── Always├── prioritize├── free├── &├── open-source├── solutions├── (VS├── Code,├── Penpot,├── Supabase/PocketBase,├── Cloudflare├── Pages).
-├── **Mobile-First**:├── Design├── and├── test├── for├── mobile├── viewport├── (390px)├── first├── before├── desktop.
-├── **Safety**:├── Never├── hardcode├── credentials,├── secrets,├── or├── API├── keys├── in├── source├── code.
```

###├── Frontend├── Scoped├── `src/app/AGENTS.md`
```markdown
#├── Frontend├── &├── Mobile├── UI├── Rules

-├── **Framework**:├── Angular├── (Signals,├── Standalone├── Components,├── control├── flow├── syntax)├── or├── modern├── reactive├── library.
-├── **Styling**:├── Tailwind├── CSS├── /├── modern├── CSS.├── Minimum├── touch├── target├── 48x48px.
-├── **Motion**:├── Compositor-only├── transforms├── (`transform`,├── `opacity`).├── Snappy├── transitions├── (150-250ms).
-├── **Typography**:├── `text-wrap:├── balance`├── for├── headings,├── `tabular-nums`├── for├── data/counters.
-├── **Safe├── Area**:├── Respect├── `env(safe-area-inset-top)`├── and├── `env(safe-area-inset-bottom)`.
-├── **PWA**:├── Maintain├── service├── worker├── caching├── and├── offline-ready├── fallback.
```

###├── Backend├── Scoped├── `src/server/AGENTS.md`
```markdown
#├── Backend├── &├── API├── Rules

-├── **Validation**:├── Strict├── schema├── validation├── for├── all├── inbound├── request├── bodies├── and├── params├── (e.g.├── Zod).
-├── **Auth**:├── Secure├── session├── cookies├── (`HttpOnly`,├── `SameSite=Lax`,├── `Secure`)├── or├── signed├── tokens.├── Never├── leak├── passwords├── or├── sensitive├── tokens.
-├── **Errors**:├── Return├── structured├── JSON├── error├── objects├── with├── machine-readable├── error├── codes.
-├── **Database**:├── All├── migrations├── committed├── to├── VCS;├── use├── parameterized├── queries├── to├── prevent├── SQLi.
```

