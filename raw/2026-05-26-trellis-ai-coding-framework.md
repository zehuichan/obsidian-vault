---
url: "https://docs.trytrellis.app/"
clipped: 2026-05-26
title: "Trellis — All-in-one AI framework & toolkit for 10+ AI coding platforms"
publisher: "Mindfold HQ"
extra_sources:
  - "https://docs.trytrellis.app/advanced/architecture"
  - "https://docs.trytrellis.app/advanced/custom-workflow"
  - "https://docs.trytrellis.app/beta/advanced/appendix-b"
  - "https://docs.trytrellis.app/advanced/appendix-f"
---

# Trellis — All-in-one AI framework & toolkit for 10+ AI coding platforms

> Beta docs cover the 0.6 track. Install latest beta: `npm install -g @mindfoldhq/trellis@beta`.

## What is Trellis?

AI's capabilities grow like vines: full of vitality but spreading everywhere. Trellis is scaffolding for AI, guiding it along the path of your conventions.

Supported platforms: Claude Code, Cursor, OpenCode, Codex, Kiro, Kilo, Gemini CLI, Antigravity, Windsurf, Qoder, CodeBuddy, GitHub Copilot, Droid, Pi Agent, plus any agent that reads the `.agents/skills/` standard (Amp, Cline, Deep Agents, Firebender, Kimi Code CLI, Warp, and more).

### One-Line Summary

**Trellis is training wheels for AI coding assistants.** It automatically injects your project specs into every AI session, so the AI writes code following your standards instead of improvising.

> AI's capabilities grow like vines, full of vitality but spreading everywhere.
> Trellis is scaffolding for AI, guiding it along the path of your conventions.

### Comparison with Traditional Approaches

| Dimension | `.cursorrules` | `CLAUDE.md` | Skills | **Trellis** |
| --- | --- | --- | --- | --- |
| Spec injection method | Manually loaded each conversation | Auto-loaded but easily truncated | User-initiated | **Auto-injection (hooks on capable platforms, prelude on others), precisely loaded per task** |
| Spec granularity | One large file | One large file | One per Skill | **Modular files, composed per task** |
| Cross-session memory | None | None | None | **Workspace journal persistence** |
| Workflow enforcement | None | None | None | **Auto-trigger skills + check sub-agent verify loop** |
| Team sharing | Single user | Single user | Shareable but no standard | **Git-versioned Spec library** |
| Platform support | Cursor only | Claude Code only | Per platform | **14 configured platforms + shared skill ecosystem** |

### Core Concepts at a Glance

| Concept | Description | Location |
| --- | --- | --- |
| **Spec** | Your coding standards, written in Markdown. AI reads specs before writing code | `.trellis/spec/` |
| **Workspace** | Each developer's session logs, letting AI remember what was done last time | `.trellis/workspace/` |
| **Task** | A work unit containing requirements docs and context configuration | `.trellis/tasks/` |
| **Skill** | Auto-triggered workflow modules: brainstorm, before-dev, check, update-spec, break-loop | platform-specific skills dir |
| **Sub-agent** | Specialized AI sub-process: `trellis-research`, `trellis-implement`, `trellis-check` | platform-specific agents dir |
| **Command** | Explicit session entries: `finish-work`, `continue`, and manual `start` where needed | platform-specific commands dir |
| **Hook** | Auto-triggered scripts that inject context at session start, sub-agent launch, etc. (platforms with hook support) | `.claude/hooks/`, etc. |
| **Journal** | Session log files recording what was done in each development session | `.trellis/workspace/{name}/journal-N.md` |

---

## Why Trellis?

| Feature | Problem Solved |
| --- | --- |
| **Auto-Injection** | Required specs and workflows auto-inject into every conversation. Write once, apply forever |
| **Auto-updated Spec Library** | Best practices live in auto-updated spec files. The more you use it, the better it gets |
| **Skill-first Workflow** | Most operations are auto-trigger skills. AI picks the right one without manual commands |
| **Team Sync** | Share specs across your team. One person's best practice benefits everyone |
| **Session Persistence** | Work traces persist in your repo. AI remembers project context across sessions |

---

## Architecture Overview (excerpt)

Trellis is a **Team-level Agent Harness with built-in LLM wiki**:

- **Agent Harness**: workflow state, hooks, skills, sub-agents, and platform adapters that control how AI coding work moves.
- **Built-in LLM wiki**: specs, tasks, research, and journals stored in the repository so AI sessions can reload project knowledge from files.
- **Team-level layer**: git-tracked workflow/spec/task files plus per-developer workspace memory, so multiple people and multiple AI tools operate against the same conventions.

### Agent Harness features

| Feature | What it does |
| --- | --- |
| Three-phase workflow | Defines Plan, Execute, and Finish in `.trellis/workflow.md`. |
| Per-turn workflow-state breadcrumb | Injects the current next-action rule into the main session on hook-capable platforms. |
| Task lifecycle | Stores requirements, status, assignee, branch, PR metadata, and subtask relationships in `.trellis/tasks/<id>/`. |
| Research / implement / check roles | Separates investigation, code writing, and verification. |
| Read-before-write enforcement | Makes implement/check paths read PRD and relevant specs before changing files. |
| Finish boundary | Separates final verification, spec update, work commit, task archive, and journal writing. |

### Built-in LLM wiki features

| Feature | What it stores |
| --- | --- |
| Spec library | Project conventions and thinking guides under `.trellis/spec/`. |
| Task knowledge | PRDs, research, info docs, and JSONL manifests under `.trellis/tasks/`. |
| Workspace memory | Per-developer journals under `.trellis/workspace/<name>/`. |
| Workflow documentation | The executable workflow contract in `.trellis/workflow.md`. |
| Local customization map | Bundled `trellis-meta` references explaining which generated files own each behavior. |

Skills cover phases where the main session needs guidance: `brainstorm`, `before-dev`, `check`, `update-spec`, `finish-work`, and meta customization. On platforms without sub-agents, skills carry more of the execution path directly in the main session.

---

## workflow.md as the executable contract

`.trellis/workflow.md` defines how Trellis runs development. Phase definitions, skill routing, per-turn reminders, and the `task.py` command catalog all live in this one file. Forking the workflow means editing one markdown file — no Python, no hook code, no re-releasing Trellis.

| Section in `workflow.md` | Who reads it | Effect |
| --- | --- | --- |
| `## Phase Index` + `## Phase 1/2/3` | AI at session start (via `SessionStart` hook) | Defines the three-phase flow and the step-by-step how-to inside each phase |
| `### Skill Routing` | AI at session start | Maps user intent → which skill to load (e.g. "wants new feature" → `trellis-brainstorm`) |
| `### DO NOT skip skills` | AI at session start | Inlines the common excuses for skipping skills and the reasons why they're wrong |
| `[workflow-state:STATUS]` blocks under `## Phase Index` | `inject-workflow-state.py` on every `UserPromptSubmit` | Per-turn reminder shown as `<workflow-state> … </workflow-state>` based on task status |
| `### Task System` (task.py command list) | AI at session start | Reference for the 16 `task.py` subcommands grouped by purpose |

The per-turn `<workflow-state>` block is what nudges the AI at each message based on the current task's `status` field. The blocks live colocated with each phase under `## Phase Index` in `workflow.md` — edit them in place. The hook script is parser-only; it never embeds fallback content of its own.

---

## Primitive table (from FAQ)

| Primitive | Triggered by | Typical use |
| --- | --- | --- |
| **Command** | User (`/trellis:*`) | Session boundaries (start, finish-work, continue) |
| **Skill** | Platform (auto-match) | Phase-level workflows (brainstorm, before-dev, check, update-spec, break-loop) |
| **Sub-agent** | Main session (spawn) | Isolated roles (implement, check, research) |

- Skill-first: `brainstorm` / `before-dev` / `check` / `update-spec` / `break-loop` moved from slash commands to auto-trigger skills that the platform matches based on user intent.

Sub-agents are available as real sub-agents on Claude Code, Cursor, OpenCode, Codex, Kiro, Gemini CLI, Qoder, CodeBuddy, Copilot, Droid, and Pi Agent. Kilo, Antigravity, and Windsurf run the same work inline in the main session. Context is hook/extension-injected on Claude Code, Cursor, OpenCode, CodeBuddy, Droid, and Pi Agent; the other sub-agent platforms use a pull-based prelude.

---

## Notes on Cursor support (from FAQ)

> Cursor doesn't have a SessionStart hook, but `.cursor/rules/trellis.mdc` is an `alwaysApply` rule that carries the Trellis prelude into every conversation. Sub-agents don't exist in Cursor, so the work that would happen in `trellis-implement` / `trellis-check` runs inline in the main session instead. Skills are delivered as rule content and matched by Cursor's rule system.

On platforms with a SessionStart hook (Claude Code, Cursor*, OpenCode, Gemini CLI, Qoder, CodeBuddy, Copilot, Droid — plus Codex if you set `codex_hooks = true`), context is injected automatically. On platforms without a SessionStart hook (Kiro, Kilo, Antigravity, Windsurf), run `/trellis:start` (or `@trellis:start` on Kiro) to load the full context report.

---

## Pages (index)

- Start Here: Trellis · Install & First Task · How It Works · Commands, Tasks & Specs · Real-World Scenarios
- Advanced: Architecture Overview · Roadmap · Customizing the Workflow · Custom Slash Commands · Custom Sub-agents · Custom Hooks · Custom Skills · Custom Spec Template Marketplace · Configuration · Multi-Platform and Team Configuration · Appendix A–F
- Use Cases: open-typeless
- Resource Marketplace: Skills · Spec Templates
- Community: Showcase · Contributing · Tech Blog · Changelog

Docs index: <https://docs.trytrellis.app/llms.txt>
