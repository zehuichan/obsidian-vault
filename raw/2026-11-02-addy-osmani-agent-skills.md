---
url: "https://github.com/addyosmani/agent-skills"
clipped: 2026-11-02
title: "Agent Skills — Production-grade engineering skills for AI coding agents"
---

# Agent Skills

**Production-grade engineering skills for AI coding agents.**

Skills encode the workflows, quality gates, and best practices that senior engineers use when building software. These ones are packaged so AI agents follow them consistently across every phase of development.

 

![Addy's Agent Skills](https://addyosmani.com/assets/images/addys-agent-skills.jpg)

```
  DEFINE          PLAN           BUILD          VERIFY         REVIEW          SHIP
 ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐      ┌──────┐
 │ Idea │ ───▶ │ Spec │ ───▶ │ Code │ ───▶ │ Test │ ───▶ │  QA  │ ───▶ │  Go  │
 │Refine│      │  PRD │      │ Impl │      │Debug │      │ Gate │      │ Live │
 └──────┘      └──────┘      └──────┘      └──────┘      └──────┘      └──────┘
  /spec          /plan          /build        /test         /review       /ship
```

---

## Commands

9 slash commands that map to the development lifecycle. Each one activates the right skills automatically.

| What you're doing | Command | Key principle |
|-------------------|---------|---------------|
| Define what to build | `/spec` | Spec before code |
| Plan how to build it | `/plan` | Small, atomic tasks |
| Build incrementally | `/build` | One slice at a time |
| Prove it works | `/test` | Tests are proof |
| Set the quality bar | `/constraints` | Decide it once, enforce it everywhere |
| Review before merge | `/review` | Improve code health |
| Audit web performance | `/webperf` | Measure before you optimize |
| Simplify the code | `/code-simplify` | Clarity over cleverness |
| Ship to production | `/ship` | Faster is safer |

In Claude Code these install as plugin commands. Where a built-in command has the same name, as `/review` does, use the namespaced form: `/agent-skills:review`.

Want fewer manual steps once the spec exists? **`/build auto`** generates the plan and implements every task in a single approved pass — you approve the plan once, then it runs autonomously. It removes the human stepping *between* tasks, not the verification: every task is still test-driven and committed individually, and it pauses on failures or risky steps.

Skills also activate automatically based on what you're doing — designing an API triggers `api-and-interface-design`, building UI triggers `frontend-ui-engineering`, and so on.

---

## Quick Start

**Fastest path — any agent, one command.** The open [skills CLI](https://github.com/vercel-labs/skills) installs into 70+ agents (Claude Code, Cursor, Codex, Copilot, Cline, and more):

```bash
npx skills add addyosmani/agent-skills            # install all 25 skills
npx skills add addyosmani/agent-skills --list     # browse before installing
```

Or grab individual skills:

```bash
npx skills add addyosmani/agent-skills --skill code-review-and-quality   # five-axis review before merge
npx skills add addyosmani/agent-skills --skill interview-me              # requirements interrogation, one question at a time
npx skills add addyosmani/agent-skills --skill test-driven-development   # red-green-refactor, enforced
```

> **Installing one skill?** A per-skill `npx` install copies only
> `skills/ /`, not the repo-level `references/` directory. The skill still
> works, but paths to supplementary shared checklists are unavailable. Use a
> whole-repo integration, clone the repository, or copy the needed checklist into
> a `references/` directory inside the installed skill. This portability gap is
> tracked in [#361](https://github.com/addyosmani/agent-skills/issues/361).

Prefer a native integration? Pick your tool below.

 
 Claude Code (recommended) 

**Marketplace install:**

```
/plugin marketplace add addyosmani/agent-skills
/plugin install agent-skills@addy-agent-skills
```

> **SSH errors?** The marketplace clones repos via SSH. If you don't have SSH keys set up on GitHub, either [add your SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) or use the full HTTPS URL to force HTTPS cloning during the marketplace-add step:
> ```bash
> /plugin marketplace add https://github.com/addyosmani/agent-skills.git
> /plugin install agent-skills@addy-agent-skills
> ```
>
> If `/plugin install` still fails with `git@github.com: Permission denied (publickey)` on Windows or macOS, the recommended workaround is to configure Git once to rewrite GitHub SSH URLs to HTTPS for subprocess clones:
> ```bash
> git config --global url."https://github.com/".insteadOf git@github.com:
> ```

**Local / development:**

```bash
git clone https://github.com/addyosmani/agent-skills.git
claude --plugin-dir /path/to/agent-skills
```

 

 
 Cursor 

Put workflow skills under `.cursor/skills/` (sync from `agent-skills/skills/`) and short policies in `.cursor/rules/*.mdc` — do not paste full skills into rules. Both directories live in your project; this repo does not ship a `.cursor/` folder. See [docs/cursor-setup.md](docs/cursor-setup.md).

 

 
 Antigravity CLI 

Install as a native plugin for skills and subagents. In affected Antigravity CLI releases, legacy command TOMLs are reported as converted but their wrapper commands are not discoverable; invoke the underlying namespaced skills directly. See [docs/antigravity-setup.md](docs/antigravity-setup.md#lifecycle-workflows-and-command-compatibility).

**Install from the repo:**

```bash
agy plugin install https://github.com/addyosmani/agent-skills.git
```

**Install from a local clone:**

```bash
git clone https://github.com/addyosmani/agent-skills.git
agy plugin install ./agent-skills
```

 

 
 Gemini CLI 

Install as native skills for auto-discovery, or add to `GEMINI.md` for persistent context. See [docs/gemini-cli-setup.md](docs/gemini-cli-setup.md).

**Install from the repo:**

```bash
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills
```

**Install from a local clone:**

```bash
gemini skills install ./agent-skills/skills/
```

 

 
 Windsurf 

Add skill contents to your Windsurf rules configuration. See [docs/windsurf-setup.md](docs/windsurf-setup.md).

 

 
 OpenCode 

Copy skills to `.opencode/skills/` (or `~/.config/opencode/skills/`), add a project-local `AGENTS.md`, and use the built-in `skill` tool for agent-driven execution. Optional slash commands can be added under `.opencode/commands/`.

See [docs/opencode-setup.md](docs/opencode-setup.md).

 

 
 GitHub Copilot 

Use agent definitions from `agents/` as Copilot personas and skill content in `.github/copilot-instructions.md`. The lifecycle slash commands (`/spec`, `/plan`, `/build`, `/test`, `/review`, `/ship`) are Claude Code commands and do not appear in Copilot; invoke the skills by name (`/spec-driven-development`, …) instead. See [docs/copilot-setup.md](docs/copilot-setup.md).

Using the standalone `copilot` CLI? Install it as a plugin — see [docs/copilot-cli-setup.md](docs/copilot-cli-setup.md).

 

 
 Kiro IDE & CLI 
 Skills for Kiro reside under ".kiro/skills/" and can be stored under Project or Global level. Kiro also supports Agents.md. See Kiro docs at https://kiro.dev/docs/skills/
 

 
 Codex 

Install as a native Codex plugin (Codex CLI v0.122+):

```bash
codex plugin marketplace add addyosmani/agent-skills
codex plugin add agent-skills@agent-skills
```

The first command registers the marketplace; the second installs the plugin. Codex reads the root `skills/` directory directly through `.codex-plugin/plugin.json`. Once installed, invoke skills in chat using `@` (e.g., `@spec-driven-development`). See [docs/codex-setup.md](docs/codex-setup.md) for local installation and troubleshooting.

 

 
 Command Code 

Install natively with the built-in `cmd skills` command. Command Code clones the repo, discovers every `SKILL.md`, and installs into `.commandcode/skills/`:

```bash
cmd skills add addyosmani/agent-skills            # pick skills to install (project)
cmd skills add addyosmani/agent-skills --global   # install for all projects (~/.commandcode/skills/)
cmd skills add addyosmani/agent-skills -s spec-driven-development  # install a specific skill
```

Installed skills show up in the TUI slash menu, e.g. `/spec-driven-development`. See [docs/commandcode-setup.md](docs/commandcode-setup.md).

 

 
 Other Agents 

Skills are plain Markdown - they work with any agent that accepts system prompts or instruction files. See [docs/getting-started.md](docs/getting-started.md). Hosts that install the pack but aren't listed above are in [docs/other-hosts.md](docs/other-hosts.md).

 

---

## Adoption

Already installed? How you roll the pack out depends on your codebase. The **[Adoption Guide](docs/adoption-guide.md)** covers two paths: the full lifecycle from day one for a greenfield project, or an incremental, verification-first rollout for an established codebase.

---

## All 25 Skills

The commands above are entry points. The pack includes 25 skills total — 24 lifecycle skills plus the `using-agent-skills` meta-skill. Each skill is a structured workflow with steps, verification gates, and anti-rationalization tables. You can also reference any skill directly.

### Meta - Discover which skill applies

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [using-agent-skills](skills/using-agent-skills/SKILL.md) | Maps incoming work to the right skill workflow and defines shared operating rules | Starting a session or deciding which skill applies |

### Define - Clarify what to build

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [interview-me](skills/interview-me/SKILL.md) | One-question-at-a-time interview that extracts what the user actually wants instead of what they think they should want, until ~95% confidence | The ask is underspecified, or the user invokes "interview me" / "grill me" |
| [idea-refine](skills/idea-refine/SKILL.md) | Structured divergent/convergent thinking to turn vague ideas into concrete proposals | You have a rough concept that needs exploration |
| [spec-driven-development](skills/spec-driven-development/SKILL.md) | Write a PRD covering objectives, commands, structure, code style, testing, and boundaries before any code | Starting a new project, feature, or significant change |
| [constraint-driven-development](skills/constraint-driven-development/SKILL.md) | Interviews you for a quality bar with sane default thresholds, writes CONSTRAINTS.md, places each check by cost, and catches agents silencing checks or skipping tests to get green | No standards are written down, or an agent is producing more than anyone reads |

### Plan - Break it down

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [planning-and-task-breakdown](skills/planning-and-task-breakdown/SKILL.md) | Decompose specs into small, verifiable tasks with acceptance criteria and dependency ordering | You have a spec and need implementable units |

### Build - Write the code

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [incremental-implementation](skills/incremental-implementation/SKILL.md) | Thin vertical slices - implement, test, verify, commit. Feature flags, safe defaults, rollback-friendly changes | Any change touching more than one file |
| [test-driven-development](skills/test-driven-development/SKILL.md) | Red-Green-Refactor, test pyramid (80/15/5), test sizes, DAMP over DRY, Beyonce Rule, browser testing | Implementing logic, fixing bugs, or changing behavior |
| [context-engineering](skills/context-engineering/SKILL.md) | Feed agents the right information at the right time - rules files, context packing, MCP integrations | Starting a session, switching tasks, or when output quality drops |
| [source-driven-development](skills/source-driven-development/SKILL.md) | Ground every framework decision in official documentation - verify, cite sources, flag what's unverified | You want authoritative, source-cited code for any framework or library |
| [doubt-driven-development](skills/doubt-driven-development/SKILL.md) | Adversarial fresh-context review of every non-trivial decision in-flight - CLAIM → EXTRACT → DOUBT → RECONCILE → STOP, with optional user-authorized cross-model escalation | Stakes are high (production, security, irreversible), working in unfamiliar code, or a confident output is cheaper to verify now than to debug later |
| [frontend-ui-engineering](skills/frontend-ui-engineering/SKILL.md) | Component architecture, design systems, state management, responsive design, WCAG 2.1 AA accessibility | Building or modifying user-facing interfaces |
| [api-and-interface-design](skills/api-and-interface-design/SKILL.md) | Contract-first design, Hyrum's Law, One-Version Rule, error semantics, boundary validation | Designing APIs, module boundaries, or public interfaces |

### Verify - Prove it works

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [browser-testing-with-devtools](skills/browser-testing-with-devtools/SKILL.md) | Chrome DevTools MCP for live runtime data - DOM inspection, console logs, network traces, performance profiling | Building or debugging anything that runs in a browser |
| [debugging-and-error-recovery](skills/debugging-and-error-recovery/SKILL.md) | Five-step triage: reproduce, localize, reduce, fix, guard. Stop-the-line rule, safe fallbacks | Tests fail, builds break, or behavior is unexpected |

### Review - Quality gates before merge

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [code-review-and-quality](skills/code-review-and-quality/SKILL.md) | Five-axis review, change sizing (~100 lines), severity labels (Nit/Optional/FYI), review speed norms, splitting strategies | Before merging any change |
| [code-simplification](skills/code-simplification/SKILL.md) | Chesterton's Fence, Rule of 500, reduce complexity while preserving exact behavior | Code works but is harder to read or maintain than it should be |
| [security-and-hardening](skills/security-and-hardening/SKILL.md) | OWASP Top 10 prevention, auth patterns, secrets management, dependency auditing, three-tier boundary system | Handling user input, auth, data storage, or external integrations |
| [performance-optimization](skills/performance-optimization/SKILL.md) | Measure-first approach - Core Web Vitals targets, profiling workflows, bundle analysis, anti-pattern detection | Performance requirements exist or you suspect regressions |

### Ship - Deploy with confidence

| Skill | What It Does | Use When |
|-------|-------------|----------|
| [git-workflow-and-versioning](skills/git-workflow-and-versioning/SKILL.md) | Trunk-based development, atomic commits, change sizing (~100 lines), the commit-as-save-point pattern | Making any code change (always) |
| [ci-cd-and-automation](skills/ci-cd-and-automation/SKILL.md) | Shift Left, Faster is Safer, feature flags, quality gate pipelines, failure feedback loops | Setting up or modifying build and deploy pipelines |
| [deprecation-and-migration](skills/deprecation-and-migration/SKILL.md) | Code-as-liability mindset, compulsory vs advisory deprecation, migration patterns, zombie code removal | Removing old systems, migrating users, or sunsetting features |
| [documentation-and-adrs](skills/documentation-and-adrs/SKILL.md) | Architecture Decision Records, API docs, inline documentation standards - document the *why* | Making architectural decisions, changing APIs, or shipping features |
| [observability-and-instrumentation](skills/observability-and-instrumentation/SKILL.md) | Structured logging, RED metrics, OpenTelemetry tracing, symptom-based alerting - instrument as you build | Adding telemetry, or shipping anything that runs in production |
| [shipping-and-launch](skills/shipping-and-launch/SKILL.md) | Pre-launch checklists, feature flag lifecycle, staged rollouts, rollback procedures, monitoring setup | Preparing to deploy to production |

---

## Agent Personas

Pre-configured specialist personas for targeted reviews:

| Agent | Role | Perspective |
|-------|------|-------------|
| [code-reviewer](agents/code-reviewer.md) | Senior Staff Engineer | Five-axis code review with "would a staff engineer approve this?" standard |
| [test-engineer](agents/test-engineer.md) | QA Specialist | Test strategy, coverage analysis, and the Prove-It pattern |
| [security-auditor](agents/security-auditor.md) | Security Engineer | Vulnerability detection, threat modeling, OWASP assessment |
| [web-performance-auditor](agents/web-performance-auditor.md) | Web Performance Engineer | Core Web Vitals audit with Quick/Deep modes and a metric-honesty rule; run it via `/webperf` |

See [docs/agents.md](docs/agents.md) for the decision matrix, orchestration rules, and how personas compose with skills and slash commands.

---

## Reference Checklists

Quick-reference material that skills pull in when needed:

| Reference | Covers |
|-----------|--------|
| [definition-of-done.md](references/definition-of-done.md) | Project-wide standing bar every change clears, contrasted with per-task acceptance criteria |
| [testing-patterns.md](references/testing-patterns.md) | Test structure, naming, mocking, React/API/E2E examples, anti-patterns (JavaScript/TypeScript) |
| [security-checklist.md](references/security-checklist.md) | Pre-commit checks, auth, input validation, headers, CORS, OWASP Top 10 |
| [performance-checklist.md](references/performance-checklist.md) | Core Web Vitals targets, frontend/backend checklists, measurement commands |
| [accessibility-checklist.md](references/accessibility-checklist.md) | Keyboard nav, screen readers, visual design, ARIA, testing tools |
| [observability-checklist.md](references/observability-checklist.md) | On-call questions, structured logging, RED/USE metrics, tracing, symptom-based alerting, pre-launch gate |
| [orchestration-patterns.md](references/orchestration-patterns.md) | Endorsed multi-persona orchestration patterns, anti-patterns, and the "personas don't invoke personas" rule |

---

## How Skills Work

Every skill follows a consistent anatomy:

```
┌─────────────────────────────────────────────────┐
│  SKILL.md                                       │
│                                                 │
│  ┌─ Frontmatter ─────────────────────────────┐  │
│  │ name: lowercase-hyphen-name               │  │
│  │ description: Guides agents through [task].│  │
│  │              Use when…                    │  │
│  └───────────────────────────────────────────┘  │                                                                                                
│  Overview         → What this skill does        │
│  When to Use      → Triggering conditions       │
│  Process          → Step-by-step workflow       │
│  Rationalizations → Excuses + rebuttals         │
│  Red Flags        → Signs something's wrong     │
│  Verification     → Evidence requirements       │
└─────────────────────────────────────────────────┘
```

**Key design choices:**

- **Process, not prose.** Skills are workflows agents follow, not reference docs they read. Each has steps, checkpoints, and exit criteria.
- **Anti-rationalization.** Every skill includes a table of common excuses agents use to skip steps (e.g., "I'll add tests later") with documented counter-arguments.
- **Verification is non-negotiable.** Every skill ends with evidence requirements - tests passing, build output, runtime data. "Seems right" is never sufficient.
- **Progressive disclosure.** The `SKILL.md` is the entry point. Supporting references load only when needed, keeping token usage minimal.

---

## Project Structure

The portable core stays in shared directories. Host-specific paths are native discovery conventions, not branding aliases; renaming or merging them would break the tools that scan those exact locations.

| Layer / consumer | Repository paths | Purpose |
|---|---|---|
| Shared workflow core | `skills/` (25 skills) | Portable `SKILL.md` workflows used by every integration |
| Shared review material | `agents/` (4 personas), `references/` (7 checklists) | Specialist reviewers and pack-level checklists carried by whole-repo installs |
| Claude Code adapter | `.claude/commands/` (9 commands), `.claude-plugin/`, `hooks/` | Slash-command wrappers, marketplace metadata, and lifecycle hooks |
| Gemini CLI adapter | `.gemini/commands/` (9 commands) | Gemini-native TOML command wrappers |
| Antigravity CLI adapter | `commands/` (9 commands), `plugin.json` | Legacy TOML wrappers and the root plugin manifest; see the [known wrapper limitation](docs/antigravity-setup.md#lifecycle-workflows-and-command-compatibility) |
| Codex adapter | `.codex-plugin/`, `.agents/plugins/` | Codex plugin metadata and marketplace registration; Codex consumes `skills/` directly |
| GitHub Copilot CLI adapter | `plugin.json` | Root plugin metadata; Copilot CLI discovers `skills/` by convention and does not register the lifecycle wrappers |
| Contributor tooling | `scripts/` (13 scripts), `evals/` (25 case files), `.github/workflows/` | Validation, routing evals, and CI |
| Documentation | `docs/` | Universal guidance and per-tool setup guides |

Tools without a checked-in adapter directory install or copy the shared `skills/` core into their own native location. The [Quick Start](#quick-start) links the setup guide for each supported host.

---

## Why Agent Skills?

AI coding agents default to the shortest path - which often means skipping specs, tests, security reviews, and the practices that make software reliable. Agent Skills gives agents structured workflows that enforce the same discipline senior engineers bring to production code.

Each skill encodes hard-won engineering judgment: *when* to write a spec, *what* to test, *how* to review, and *when* to ship. These aren't generic prompts - they're the kind of opinionated, process-driven workflows that separate production-quality work from prototype-quality work.

Skills bake in best practices from Google's engineering culture — including concepts from [Software Engineering at Google](https://abseil.io/resources/swe-book) and Google's [engineering practices guide](https://google.github.io/eng-practices/). You'll find Hyrum's Law in API design, the Beyonce Rule and test pyramid in testing, change sizing and review speed norms in code review, Chesterton's Fence in simplification, trunk-based development in git workflow, Shift Left and feature flags in CI/CD, and a dedicated deprecation skill treating code as a liability. These aren't abstract principles — they're embedded directly into the step-by-step workflows agents follow.

---

## How it compares

Wondering how this stacks up against [Superpowers](https://github.com/obra/superpowers) or [Matt Pocock's skills](https://github.com/mattpocock/skills)? See **[docs/comparison.md](docs/comparison.md)** for an honest, side-by-side look at how the three are shaped differently and when to reach for each — including a link to a controlled [head-to-head experiment](https://www.linkedin.com/pulse/superpowers-vs-agent-skills-faster-shipping-safer-reasoning-om-mishra-dzakf/).

---

## Contributing

Skills should be **specific** (actionable steps, not vague advice), **verifiable** (clear exit criteria with evidence requirements), **battle-tested** (based on real workflows), and **minimal** (only what's needed to guide the agent).

See [docs/skill-anatomy.md](docs/skill-anatomy.md) for the format specification, [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines, and [docs/advanced-per-agent-configuration.md](docs/advanced-per-agent-configuration.md) for applying model routing, tool restrictions, and other per-agent runtime controls without breaking the portable `SKILL.md` frontmatter.

---

## Team

agent-skills is built and maintained by:

| | Name | GitHub | Role |
|---|------|--------|------|
| | **Addy Osmani** | [@addyosmani](https://github.com/addyosmani) | Creator |
| | **Federico Bartoli** | [@federicobartoli](https://github.com/federicobartoli) | Collaborator |
| | **Joan León** | [@nucliweb](https://github.com/nucliweb) | Collaborator |

---

## License

MIT - use these skills in your projects, teams, and tools.


---

# Appendix: docs/comparison.md

Source: https://github.com/addyosmani/agent-skills/blob/main/docs/comparison.md

<!--
  This document is for developers evaluating the project. It is NOT a skill and
  is not meant to be loaded into an agent's context. It lives in docs/ so it
  stays out of the agent's working set.
-->

# How agent-skills compares

People often ask how **agent-skills** relates to the two other "skills for coding agents" collections they hear about most: **Superpowers** (Jesse Vincent / obra) and **Matt Pocock's skills**. All three are good, share a lot of DNA, and are worth learning from. This page is an honest map of how they are *shaped* differently, so you can pick the one that fits how you work, or borrow across all three.

> **TL;DR** — They optimize for different moments. **agent-skills** organizes the *whole product lifecycle* (Define, Plan, Build, Verify, Review, Ship) with review personas, anti-rationalization guards, and an in-repo eval framework that checks the skills actually route and behave. **Superpowers** leans into *autonomous, reasoning-heavy* runs with subagents, a strict pipeline, and worktree isolation. **Matt Pocock's skills** are a *sharp, opinionated Claude Code toolkit* distilled from one expert's daily workflow, with a signature "grill me" interrogation loop. None is "best" in the abstract; it depends on the work in front of you.

---

## At a glance

| | **agent-skills** | **Superpowers** | **Matt Pocock's skills** |
|---|---|---|---|
| **Core idea** | Encode the full senior-engineering lifecycle as skills | A complete development *methodology* built on composable skills | One expert's Claude Code workflow, open-sourced and evolving |
| **Organizing principle** | SDLC **phases** (Define to Ship) behind a meta-skill router | A single disciplined loop: brainstorm, plan, execute, review | A curated toolbox of focused, composable commands |
| **Catalog size** | 25 skills spanning the whole lifecycle | ~14 skills, deep on the inner build loop | ~30 skills, grouped into engineering / productivity / in-progress / deprecated |
| **Lifecycle coverage** | Broad: idea refinement, API and UI design, security, performance, CI/CD, observability, deprecation, ADRs, launch | Deep but narrow: TDD, debugging, planning, review, skill authoring | Define and Build heavy: grilling, PRDs, issues, TDD, architecture, bug triage, knowledge management |
| **Entry points** | Slash commands mapped 1:1 to phases (`/spec` `/plan` `/build` `/test` `/review` `/code-simplify` `/ship`, plus `/webperf`), with a `/build auto` full-plan mode | Skill-chained pipeline (`brainstorming`, `writing-plans`, `subagent-driven-development`) | Slash commands (`/grill-me`, `/tdd`, `/to-prd`, `/diagnosing-bugs`, `/grill-with-docs`) |
| **Distinctive mechanisms** | Anti-rationalization tables and Red Flags in every skill; parallel review **personas** in `/ship`; reference checklists; a **three-tier eval framework** in CI | Subagent-driven development with a task reviewer (spec + quality) and a fix loop; git-worktree isolation; skills-that-write-skills, pressure-tested | The **grilling** primitive (one question at a time, design-tree walking); seam-based TDD; explicit user-invoked vs model-invoked split; issue-tracker integration |
| **Quality measurement** | Trigger, routing, and behavioral evals run against the catalog (in-repo, some in CI) | Pressure-testing methodology is core to its philosophy; the eval suite itself now lives in a separate repo | None shipped in-repo |
| **Tooling reach** | Claude Code, Cursor, Gemini CLI, Antigravity, OpenCode, Windsurf, Copilot, Kiro, Codex, Command Code, plus the `npx skills` CLI | One of the widest and most actively churned surfaces: Claude Code, Codex, Cursor, Copilot CLI, OpenCode, Kimi, Factory Droid, Antigravity, Pi | Claude Code first, distributed via `npx skills add`; other agents work with varying fidelity |
| **Governance** | Actively reviews and merges community contributions; every skill ships an eval | Largely solo-authored; a substantial backlog of unmerged community PRs | Solo-authored, self-merged, developed openly in public |
| **Best for** | Driving a feature through every phase with a human checkpoint at each | Long, autonomous, reasoning-heavy or exploratory work | A pragmatic, battle-tested daily loop, strongest at requirements and TDD |

*(We deliberately leave out star counts and adoption figures: they are cited wildly differently across blogs and change weekly. All three are actively used and maintained.)*

---

## The three projects, in their own terms

### Superpowers (obra)

A full software-development *methodology* built on composable skills. It bets on **autonomy and upfront reasoning**. A session runs a deliberately linear pipeline: Socratic brainstorming that writes a dated spec and permits exactly one handoff, detailed plans written to be executable by "an enthusiastic junior engineer with poor taste and no context," then subagent-driven execution where a fresh subagent implements each task and a task reviewer signs off on both spec compliance and code quality before it closes, with a fix loop for anything it flags and a whole-branch review on the most capable model at the end. Git worktrees keep parallel work isolated, and its `writing-skills` skill applies TDD to documentation itself: no skill ships without a failing test first.

Its strengths are real: hand off a sizable chunk and come back to a reviewed result, with strong guardrails against an agent rationalizing its way past the process. The trade-offs are the flip side of that discipline. Coverage is narrow (it is an inner-loop build methodology, not a security-to-launch lifecycle), the single pipeline can feel heavy on small changes, and the recent direction has been about paying down that cost, for example consolidating to a single task reviewer for roughly twice the speed and half the tokens. The loudest thing its community keeps asking for, multi-agent team execution, is not yet in the box.

**Repo:** <https://github.com/obra/superpowers>

### Matt Pocock's skills

Matt open-sourced the Claude Code skills he actually uses, and the collection has grown into a broad, opinionated toolbox organized in public, complete with visible `in-progress/` and `deprecated/` directories that model the discipline the skills preach. The center of gravity is not TDD, it is **grilling**: a reusable interrogation loop that asks one question at a time, walks each branch of the design tree resolving dependencies in order, offers a recommended answer per question, prefers reading the codebase over asking, and refuses to proceed until you confirm a shared understanding. `grill-me` and `grill-with-docs` are thin wrappers over it. The TDD skill is distinctive too and cheerfully heterodox ("refactoring is not part of the loop," it belongs to code review), and there is a deliberate split between user-invoked and model-invoked skills to treat agent context as a scarce budget.

Its strengths are authenticity and sharpness: this is how one very good engineer ships, not a committee's idea of a framework, and the requirements-grilling loop is genuinely excellent. The trade-offs: it is Claude Code first in practice (other agents surface reliability rough edges), there are no in-repo evals to catch regressions, lifecycle coverage past Build is thin, and some skills couple to a personal setup wizard and tracker conventions. Recent work is pushing from single-session skills toward multi-session orchestration through issue trackers (the in-progress `wayfinder`).

**Repo:** <https://github.com/mattpocock/skills> · related: <https://github.com/mattpocock/agent-rules-books>

### agent-skills (this project)

agent-skills organizes the **entire product lifecycle** as skills, with a meta-skill (`using-agent-skills`) that routes a task to the right one. Every skill carries a **Common Rationalizations** table (the excuses an agent makes to skip a step, each rebutted) and **Red Flags**. Slash commands map one-to-one to lifecycle phases; `/build auto` runs a whole approved plan in one pass; and `/ship` fans out review **personas** (`code-reviewer`, `security-auditor`, `test-engineer`, `web-performance-auditor`) in parallel, then merges them into a go/no-go. It keeps a human checkpoint at each phase, ships seven reference checklists including a Definition of Done, and runs across most major agent tools with a single-command install on several of them.

What is newer, and the current point of difference: a **three-tier eval framework** lives in the repo. Tier 1 checks structure, Tier 2 checks that each skill's description carries the vocabulary users actually say and that no two skills collide on routing (deterministic, runs in CI), and Tier 3 grades an agent's real execution trace against per-skill expectations. Neither of the other two ships that kind of in-repo, catalog-wide measurement today. The honest trade-off in the other direction: agent-skills has less of a single opinionated "run" than Superpowers, and none of the three has yet solved durable cross-session memory well.

---

## A real head-to-head: Superpowers vs. agent-skills

Om Mishra ran a controlled experiment, same model, same repo, same prompt in Claude Code, changing only the skill framework, and wrote it up:

**["Superpowers vs Agent-Skills: Faster Shipping, Safer Reasoning"](https://www.linkedin.com/pulse/superpowers-vs-agent-skills-faster-shipping-safer-reasoning-om-mishra-dzakf/)** by Om Mishra

Summarized fairly:

- **agent-skills** moved to code faster (~8 min vs ~12) and ran **more validation passes** (7 vs 5, including the full test suite). That broader validation caught a compatibility issue *outside* the immediate feature that the feature-specific tests missed. He gave it the edge on **validation depth** for that task.
- **Superpowers** invested more **upfront architectural reasoning**, which he still prefers as his daily driver for evolving production systems and exploratory work with no established pattern to follow.
- Token efficiency was effectively identical; both replanned once.

It is one developer's single-task experiment, not a benchmark, but it is a concrete illustration of the core trade-off: **broad disciplined validation versus heavy upfront reasoning**. His own conclusion is the honest one: pick the tool to the task.

---

## How to decide what to use

The at-a-glance table tells you how they are shaped. This is how to choose in practice.

### Start with the shape of your work

- **A whole feature, front to back?** agent-skills. It is the only one of the three that carries you from spec through security, performance, and launch with a checkpoint at each phase, so nothing quietly skips the review or the pre-flight.
- **A big, ambiguous chunk you want to hand off and walk away from?** Superpowers. Its pipeline and subagent review are built to run for a long time and hand back a result that has already been reviewed against the spec.
- **A fast, focused daily loop, especially getting requirements right before code?** Matt Pocock's skills. The grilling loop is the sharpest requirements tool of the three, and the toolkit stays out of your way.

### Then weight what you actually care about

- **Breadth of coverage** (security, performance, CI/CD, observability, launch): agent-skills is the clear pick; the others are inner-loop focused.
- **Autonomy over a long run**: Superpowers, by design.
- **Low ceremony on small changes**: Pocock's toolkit is lightest; agent-skills offers a middle gear (a small change can skip straight to `/test` and `/review`); Superpowers is the most process-heavy.
- **Confidence that the skills themselves work**: agent-skills is the only one with catalog-wide evals in the repo, so a description or routing regression fails CI rather than surfacing as a mysterious "why didn't the skill fire" later.
- **Requirements interrogation**: Pocock's grilling is the reference implementation; agent-skills' `interview-me` is close in spirit and gaining an opt-in collaborative mode.
- **Platform spread**: agent-skills and Superpowers both run almost everywhere; Pocock is happiest on Claude Code.
- **A human gate at each step vs. a hands-off run**: agent-skills checkpoints by default; Superpowers minimizes mid-run check-ins on purpose.

### Concrete scenarios

- *"Ship a new endpoint with auth, tests, and a security pass before merge."* agent-skills: `/spec` to `/ship`, with the security-auditor and test-engineer personas fanning out at the end.
- *"Refactor a gnarly subsystem overnight and review it in the morning."* Superpowers: hand it the plan, let subagent-driven development and the task reviewer run.
- *"I have a vague idea and keep letting the agent guess the requirements."* Pocock's `grill-me` (or agent-skills' `interview-me`) to pin down intent before any code.
- *"Fix one clear bug, test-first."* Any of them; reach for the lightest one you already have installed.
- *"Standardize how a team of engineers uses agents across a repo."* agent-skills: the phase commands, personas, and shared checklists give a team a common vocabulary, and the evals keep custom skills honest.
- *"Roll my own skills and know they trigger correctly."* agent-skills, and borrow Superpowers' pressure-testing discipline for the ones that must not be rationalized away.

### Solo vs. team

For a solo developer, taste and momentum win: pick the one whose defaults match how you already work, and the lowest-ceremony option is often the most-used one. For a team, the calculus shifts toward shared structure and guardrails you can enforce: named phases, review personas, checklists, and evals that fail CI are what keep five people (and their agents) converging on the same standard rather than five private workflows. That is the situation agent-skills is built for, though a team can absolutely adopt Superpowers' review discipline or Pocock's grilling loop within it.

### The shared frontier (true of all three)

None of these has solved **durable cross-session memory** well yet: what an agent learned in one session rarely carries cleanly into the next. All three are circling it (learnings files, handoff artifacts, tracker-backed planning maps). If that is your bottleneck, know that you are at the edge of what any of them ships today, and expect to stitch some of it yourself for now.

---

## Combining them

You do not have to choose exclusively, but combine with care. These are Markdown skills, not runtimes, so cherry-picking *individual* skills works well: pull in Pocock's `grill-me`, a Superpowers isolation pattern, or a specific checklist alongside your main setup.

What does not work is running two of them as your **active router at the same time**. Stacked meta-skills fight over command names (`/tdd` defined in two places), compete on routing logic, and pull in different TDD philosophies, so you get unpredictable behavior rather than the best of both. Pick one framework as your primary router, and borrow from the others a la carte.

---

## Sources

- Superpowers: <https://github.com/obra/superpowers>
- Matt Pocock's skills: <https://github.com/mattpocock/skills>
- Om Mishra, *Superpowers vs Agent-Skills*: <https://www.linkedin.com/pulse/superpowers-vs-agent-skills-faster-shipping-safer-reasoning-om-mishra-dzakf/>

*Spotted something inaccurate about another project here? Open an issue or PR. We would rather be fair than flattering.*

