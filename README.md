# SDD Cursor — Spec-Driven Development for Cursor IDE

> A lightweight, opinionated framework for AI-assisted backend development
> with a clear separation of concerns: **design first, build with TDD,
> review before commit**.

[![Cursor](https://img.shields.io/badge/Cursor-IDE-black?logo=cursor)]()
[![License](https://img.shields.io/badge/license-MIT-blue.svg)]()
[![Status](https://img.shields.io/badge/status-stable-brightgreen.svg)]()

---

## Why this exists

AI-assisted coding has a known failure mode: a single agent does too many
things at once — designs, implements, tests, and reviews — and the result
is plausible code that misses the architecture you actually wanted.

This framework solves it by splitting the work across **three personas**
running in sequence, coordinated by **five slash commands** in Cursor:

| Persona | Command | Job |
|---------|---------|-----|
| 🧭 Architect | `/design` | Translates intent into a four-file spec, decomposed by layer |
| 🛠 Developer | `/build` | Implements ONE mini-spec with TDD, then stops |
| 🔍 Reviewer | `/review` | Audits the implementation, approves or returns a diff |
| 🧪 (utility) | `/test` | Runs test suites independently |
| ✨ (utility) | `/refactor` | Quality pass: phpcs + phpstan |

Each persona is enforced via a dedicated `SKILL.md` and is **forbidden**
from doing the other personas' work. The architect doesn't write code.
The developer doesn't auto-advance. The reviewer doesn't fix code, only
flags. That separation is what makes the loop converge instead of drift.

---

## Quick start

```bash
# 1. Apply the framework to your project
npx degit [ivan-rdz/sdd-cursor/symfony-php/template](https://github.com/PortilloDev/spec-driven-development-framework) .cursor

# 2. Personalize stack and conventions (pending implement)
bash .cursor/bootstrap.sh

# 3. In Cursor, design your first feature
/design my-first-feature

# 4. Implement layer by layer
/build @.cursor/specs/my-first-feature/tasks/01-domain.md
/review

# 5. Repeat for each layer
/build @.cursor/specs/my-first-feature/tasks/02-application.md
/review
# ...
```

---

## Available stacks

| Stack | Path | Status |
|-------|------|--------|
| Symfony 6.4 / PHP 8.3 + Hexagonal + CQRS | `symfony-php/template` | ✅ Stable |
| FastAPI / Python 3.11 + Hexagonal | `fastapi-py/template` | 🚧 In progress |
| Laravel 11 + Modular Monolith | `laravel-php/template` | 📋 Planned |

Pick the variant that matches your project. The workflow is identical
across stacks; only the rules and skill knowledge change.

---

## What you get

After running `degit` and `bootstrap.sh`, your project has:

````
.cursor/
├── README.md                       # Local guide for your team
├── commands/
│   ├── design.md                   # Slash command: /design
│   ├── build.md                    # Slash command: /build
│   ├── review.md                   # Slash command: /review
│   ├── test.md                     # Slash command: /test
│   └── refactor.md                 # Slash command: /refactor
├── rules/
│   ├── workflow.mdc                # Loaded always — pipeline reference
│   ├── backend-standards.mdc            # Loaded always — architectural contract
│   └── stack.mdc                   # Loaded always — tech stack & tooling
├── skills/
│   ├── architect/SKILL.md          # Persona: design without coding
│   ├── developer/SKILL.md          # Persona: TDD-driven, single task
│   └── senior-reviewer/SKILL.md    # Persona: audit, never modify
└── specs/
└── template/                   # Spec scaffolding used by /design
````
Plus a `bootstrap.sh` that asks you a few questions and tailors the rules
to your project (project name, languages, async transport, etc.).

---

## How a feature flows

┌─────────────────────────┐
               │     /design feature     │
               │   (architect persona)   │
               └────────────┬────────────┘
                            │
                            ▼
          .cursor/specs/feature-name/
          ├── 00-global-spec.md       What & Why
          ├── 00-technical-plan.md    How
          ├── tasks/
          │   ├── 01-domain.md        ≤ 2h, single commit
          │   ├── 02-application.md   ≤ 2h, single commit
          │   ├── 03-infrastructure.md ≤ 2h, single commit
          │   └── 04-integration.md   ≤ 2h, single commit
          └── status.md               Progress tracker

                            │
                            ▼
               ┌─────────────────────────┐
               │   /build {task-file}    │
               │   (developer persona)   │
               │     red → green         │
               │     run tests           │
               │     stop                │
               └────────────┬────────────┘
                            │
                            ▼
               ┌─────────────────────────┐
               │        /review          │
               │   (reviewer persona)    │
               │     run gates           │
               │     decide              │
               └────────────┬────────────┘
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
        APPROVED                  CHANGES REQUESTED
        next task                 /build same task
                                  (fixing only flagged issues)

                                  ---

## Design principles

### 1. Decompose before you build

Large specs cause hallucinations. Each feature is split into
**single-commit, ≤ 2h, single-layer mini-specs**. The architect cannot
emit a task larger than that — it has to split.

### 2. Personas have hard boundaries

The skills don't suggest what each persona should do — they enforce it
with explicit "NEVER" rules. The developer cannot pick the next task.
The reviewer cannot edit code. This is the discipline that makes the
loop work.

### 3. TDD is non-optional

Every public class/method is born from a failing test. `/build` will
refuse to mark a task as "ready for review" if any test is red. This
isn't dogma — it's the only way to verify the spec was implemented and
not approximated.

### 4. The contract is in `.mdc`

`backend-standards.mdc` is the source of truth for layer rules, naming,
CQRS contracts. The reviewer enforces it line by line. If you want
to change the contract, you change the `.mdc` — never the code.

### 5. Light context, heavy skills

`alwaysApply: true` rules stay short (~150 lines combined). Detail
lives in skill files (`SKILL.md`) that load only when invoked. This
keeps every conversation's context window for actual work, not boilerplate.

---

## Comparison with other tools

| Tool | Approach | Best for |
|------|----------|----------|
| **OpenSpec** | Generic spec format with CLI lifecycle | Multi-tool teams, language-agnostic |
| **Spec-Kit / Kiro** | Heavy upfront design, full plan before code | Greenfield projects with stable scope |
| **BMAD-METHOD** | Multi-agent simulation in Cursor | Teams comfortable with agent orchestration |
| **SDD Cursor (this)** | Three-persona pipeline, TDD-enforced, stack-specific | Brownfield + greenfield, opinionated stacks, solo or small teams |

The differentiator is **opinionatedness**: SDD Cursor knows your stack
(Symfony+CQRS, FastAPI+Hexagonal, etc.) and bakes that knowledge into
the skills. Generic tools can't do that without configuration.

---

## When to use this — and when not to

✅ **Good fit**:
- Projects with a clear architectural contract (Hexagonal, Clean, DDD)
- Backend work where layer separation matters
- Brownfield codebases where consistency is more valuable than speed
- Solo developers or small teams using Cursor as primary IDE
- Stacks where one of the variants applies

⚠️ **Bad fit**:
- One-shot scripts or throwaway prototypes
- Frontend-heavy projects (this framework is backend-shaped)
- Teams using Claude Code or Aider as primary tool (the slash commands
  are Cursor-specific)
- Projects where you want the AI to make architectural decisions
  autonomously

---

## Customizing your setup

The framework is meant to be a starting point, not a cage. Common
customizations:

### Add your own architectural rules

Drop a new file in `.cursor/rules/`:

```markdown
---
description: Project-specific business rules
alwaysApply: true
---

# {Your Project} business rules
- All money values stored as integer cents
- Audit log mandatory on entity mutation
- ...
```

### Tweak a persona

Edit `.cursor/skills/{persona}/SKILL.md` directly. The framework's
defaults are guidelines; if your team has a stricter review checklist,
add it to the senior-reviewer skill.

### Add a new layer task type

Default mini-specs are `01-domain`, `02-application`, `03-infrastructure`,
`04-integration`. If your stack has more layers (e.g. `02b-events.md`
for event handlers), the architect already supports splitting — just
mention it in your `/design` request.

---

## Updating an existing project

To pull updates from upstream:

```bash
# Backup your local customizations
cp -r .cursor .cursor.bak

# Apply the new version
npx degit ivan-rdz/sdd-cursor#v1.2.0/symfony-php/template .cursor --force

# Diff and merge your customizations back in
diff -ruN .cursor.bak .cursor
```

Or pin to a tag and update intentionally:

```bash
npx degit ivan-rdz/sdd-cursor#v1.2.0/symfony-php/template .cursor
```

---

## FAQ

**Why not OpenSpec or Spec-Kit?**
Both are excellent, but generic. After using OpenSpec on a Symfony+CQRS
project, the generated code consistently missed our naming conventions,
async patterns, and domain event recording. Generic spec formats can't
encode stack-specific contracts. This framework can — at the cost of
being narrower in scope.

**Why personas instead of one big prompt?**
A single prompt with "be an architect, then a developer, then a reviewer"
collapses into the strongest of the three. Separate skill files with
explicit `NEVER` rules don't. The model literally has different behavior
when each skill is loaded as the active persona.

**Does this work in Claude Code or Aider?**
The slash commands are Cursor-specific. The skills (`SKILL.md` format)
are compatible with Claude Code. With minor adaptations the rules and
skills can be reused; the commands need to be re-expressed as Claude
Code slash commands or Aider scripts. PRs welcome.

**Can I use this for frontend?**
The framework is shaped for backend work (layers, CQRS, repositories).
Frontend has different decomposition axes (component, store, route).
A `react-ts/template` variant is on the roadmap but not started.

**Is this production-tested?**
It's used daily on a Symfony 6.4 + Doctrine codebase with ~26k LOC of
monthly batch processing, plus a couple of greenfield projects. That's
not "battle-tested at scale" but it's not a weekend prototype either.

---

## Roadmap

- [ ] FastAPI / Python variant (`fastapi-py/template`)
- [ ] Laravel 11 variant (`laravel-php/template`)
- [ ] Pre-commit hook to validate `status.md` against actual test results
- [ ] CLI replacement for `bootstrap.sh` with proper interactive prompts
- [ ] Example feature gallery (`examples/`) covering CRUD, async, events
- [ ] Compatibility layer for Claude Code slash commands

See [open issues](https://github.com/PortilloDev/spec-driven-development-framework/issues) for the
full list.

---

## Contributing

Contributions welcome, especially:
- New stack variants (open an issue first to align on scope)
- Improvements to the personas (`SKILL.md` files)
- Real-world example features for `examples/`

For larger changes, please open an issue describing the motivation
before submitting a PR.

---

## License

MIT — use it, fork it, ship it.

---

## Credits & inspiration

This framework stands on shoulders:

- [OpenSpec](https://github.com/openspec/openspec) — for the
  spec/changes/archive lifecycle pattern
- [Spec-Kit](https://github.com/github/spec-kit) and
  [Kiro](https://kiro.dev) — for spec-driven development as a discipline
- [BMAD-METHOD](https://github.com/bmadcode/BMAD-METHOD) — for the
  multi-agent persona idea
- The Cursor and Claude Code teams — for making slash commands and
  skills first-class citizens

The opinionated parts (TDD enforcement, layer-by-layer decomposition,
strict persona boundaries) come from real-world pain on a Symfony+DDD
codebase that taught us what generic frameworks miss.

---

**Made by [Iván Portillo Pérez](https://ivanportilloperez.com)** —
backend developer, podcast host of *Artesanos del Código*, occasionally
shipping things that work.
