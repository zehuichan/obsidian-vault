---
url: "https://github.com/langchain-ai/design-agent"
clipped: 2026-10-29
title: "langchain-ai/design-agent: Open-source design agent (Content Studio) built on Deep Agents"
---

# langchain-ai/design-agent

> Open-source design agent: on-brand stills, decks, diagrams, charts and motion graphics from a chat thread, built on Deep Agents.

**仓库元数据（剪藏时，GitHub API）**：354 stars · 47 forks · 1 open issue · License Apache-2.0 · 主语言 JavaScript（实际为 Python + JS 混合）· 创建于 2026-10-18 · 最近 push 2026-10-27 · 默认分支 `main`

**Topics**：agent · agents · ai-agent · deepagents · design · hyperframes · langgraph · managed-agents · motion-graphics · paper · slack · video

---

## README

Distribute Content Studio to your team → · Try it locally, no keys needed · How we built it · Docs

**Content Studio is the design agent LangChain runs, now open source.** Add your brand and deploy it to your team's Slack with Managed Deep Agents. Anyone can ask it like they'd ask a designer, attach what they have, and get finished, on-brand files back, from a resized ad to a launch film.

Since August it has shipped over 3,000 assets for us, posts with its work drew about 4.5M views on X, and three in four requests come from outside marketing. [How we built it](https://www.langchain.com/blog/design-agent) goes deeper into each part.

### See it work

We asked Content Studio for its own launch film. It went full Rocky.

（视频：https://github.com/user-attachments/assets/ee5b98e0-7ec6-4b24-ada6-1f27087947a3）

From a 1-bit chick to champion in 47 seconds.

### How it works

Content Studio is a [Deep Agents](https://github.com/langchain-ai/deepagents) agent deployed on [Managed Deep Agents](https://docs.langchain.com/langsmith/python/managed-deep-agents-overview). Managed Deep Agents runs the infrastructure: Slack, sandboxes, memory, schedules and tracing. This repo is the design agent on top. The model makes the creative calls, and code renders, checks and delivers.

How we run it at LangChain. Your copy starts with the integrations you turn on (Integrations).

- **Slack.** Mention it in a thread with a brief and any files. MDA's Slack app hands the message and attachments to the agent, and the finished files come back to the same thread. While it works, one message updates step by step, a footer shows what's running and the cost so far, and anything you add mid-job reaches the work in progress.
- **A computer per thread.** Each thread gets its own LangSmith sandbox: a Linux machine with Chrome, FFmpeg, [HyperFrames](https://hyperframes.heygen.com), D2 and your fonts, built from one pinned recipe ([`sandbox/setup.sh`](sandbox/)). Everything it makes is code in a project folder there (a slide is HTML, a film is an animation timeline), so a change is an edit and a re-render, not a restart. API keys never enter it.
- **Skills and a design wiki.** The always-on prompt is small. For each ask it first reads a router skill that picks the right workflow, from a resized ad to a launch film, out of 35 [skills](skills/). The wiki holds the craft (type, colour, layout, charts) and your brand pack holds the facts. Skills load only when a job needs them, served by MDA from Context Hub.
- **Your tools.** Video is built in HyperFrames, which treats a film as a web page on a timeline. [Paper](https://paper.design) is a shared canvas where you edit a storyboard or layout directly. Image, voice and music models run as tools outside the sandbox, and MCP servers like Notion, Linear, GitHub and Figma connect through MDA, with each person signed in as themselves.
- **Async subagents.** On a big job the main agent plans, splits the work and starts subagents in the background on the same sandbox and files. MDA wakes it when one finishes, fails or stalls, so it never sits waiting and can keep talking to you. Films go storyboard, animatic, then production, and code makes every part pass review before assembly.
- **Memory.** Tell it a rule once ("never use orange on LinkedIn") and it saves it that turn. MDA gives it two layers of files: team memory everyone shares and personal memory for each person, and each layer's index loads into every run.
- **It gets better.** After a job, `learn_from_job` writes notes on what worked against the skill it used. Later jobs read and rate those notes, and the ones that don't hold up are dropped.

#### Watch it explain itself

We asked it to explain its own harness as a nature documentary, starring a pixel parrot. It walks through the computer, the tools and the context behind its work.

（视频：https://github.com/user-attachments/assets/65eac8e4-0fac-4a5a-9cca-02e26a0a8baa）

Storyboarded, built and narrated by the agent, in one Slack thread.

**The prompt that made this film**

The agent made it from this ask and a few rounds of notes:

> Make a short explainer of how you create videos in code, end to end, for our blog post on the media harness we built on Deep Agents. Narrate it like a David Attenborough nature documentary, starring the pixel parrot from our promo, with your subagents as its flock. Keep it technical enough for an engineer, in real terms: skills, wiki, tools, apps. Storyboard it before you build, and end on "Built on Managed Deep Agents".

Read the [architecture](docs/architecture/README.md), or start changing it: [instructions.md](instructions.md) · [skills/](skills/) · [Customization](docs/customization.md) · [Evals](docs/evals.md) · [Operations](OPERATIONS.md) · [Contributing](CONTRIBUTING.md).

### Why it doesn't look like AI

The same prompt to a local coding agent with a brand skill, and to Content Studio.

- **It looks at its work.** It renders, then looks at the image before it posts anything.
- **Your brand is code.** Layouts only take your tokens: your colours, fonts and spacing.
- **It checks what a machine can count.** Colours, fonts, contrast, sizes and chart math are checked in code on every render. Taste stays with your designers.

### Quickstart

| Building it yourself | Starting here |
| --- | --- |
| Months on a harness, a sandbox, skills and a Slack UX | Fork it, add your brand, deploy with one command |
| Polished work that looks like a hundred others | Work that follows your brand, checked on every render |
| Design requests wait in a queue | Anyone asks in Slack: a graphic in minutes, a long deck overnight |
| Every team fixes the same mistakes | Fix a skill once, and every asset after it follows |

This repository is a [Managed Deep Agents](https://docs.langchain.com/langsmith/python/managed-deep-agents-overview) project, so it deploys to your LangSmith workspace as it is. You need a LangSmith account (Managed Deep Agents is in public beta on LangSmith Cloud) and a model key:

```bash
curl -fsSL https://raw.githubusercontent.com/langchain-ai/design-agent/main/install.sh | bash
cd design-agent
uv run design-agent config set LANGSMITH_API_KEY=lsv2_… ANTHROPIC_API_KEY=sk-ant-…
uv run design-agent sandbox publish   # once per workspace: the agent's computer
uv run mda deploy
```

The installer gets [uv](https://docs.astral.sh/uv/) if you don't have it, clones the repository and installs its dependencies; `git clone` and `uv sync` do the same.

The first deploy sets up the agent's Slack app. Authorize it in your workspace, then mention the agent in a channel. It's how we run ours at LangChain. Try changes on a separate copy with `uv run mda deploy --name design-agent-test`; [Deploy with MDA](docs/deploy-mda.md) covers keys, what ships and checking it works.

**Or ask your coding agent.** Open the repo in Claude Code, Codex or Cursor and say what you want. [AGENTS.md](AGENTS.md) routes each request to a step-by-step [skill](.agents/skills/):

| Say | It will |
| --- | --- |
| "Deploy it to our Slack." | publish the sandbox, check what ships, and deploy after your yes |
| "Build our brand pack from our website and brand guide." | draft the pack, validate it, and ask you to review it |
| "Teach it to make our weekly newsletter header." | add a skill for it and try it on a real request |
| "Did that change make it better?" | run the evals before and after, and show you the numbers |

### Try it locally

The demo runs the real agent, sandbox and checks with a scripted model and no API keys. It needs **Docker**, and leaves a launch card in three sizes and a 4-second teaser in `workspace/out/demo/`:

```bash
uv run design-agent sandbox build
uv run design-agent demo
```

With a model key in `.env`, ask for something real. Files land in `workspace/out/<thread>/`, and the last line tells you what the run cost:

```bash
uv run design-agent ask "A LinkedIn card announcing our design review service, 1200x627."
```

### Make it yours

| Want | Add | Guide |
| --- | --- | --- |
| Your brand | `workspace/brand/` (until then, a neutral sample brand) | [brand-onboarding skill](.agents/skills/brand-onboarding/SKILL.md) |
| Generated images | `OPENAI_API_KEY` | [Models](docs/models.md) |
| Voice, music, sound effects | `ELEVENLABS_API_KEY` | [sound skill](skills/sound/SKILL.md) |
| The Paper canvas | `PAPER_MODE=local` with Paper Desktop, or `bridged` when deployed | [Paper](docs/partners/paper.md) |
| Web search, your docs, other tools | MCP servers in `config/mcp.toml` | [Customization](docs/customization.md) |

Everything on this page uses LangChain's brand, because that's the one we run. Licensed fonts go in `private-fonts/`, which Git ignores.

### Questions

**What does it cost?** The code is free (Apache 2.0). You pay your model provider: at LangChain the median request costs about 50 cents in model spend, and long films and decks cost more. Managed Deep Agents [hosting is paid](https://www.langchain.com/pricing); it runs the agent, the sandboxes, the Slack app and the schedules that you'd otherwise build and operate yourself.

**Will it use my brand?** Yes. Add your brand pack and every file follows it. The [brand-onboarding skill](.agents/skills/brand-onboarding/SKILL.md) builds the pack with you from your brand guide, website, files or a Paper file.

**Where do I start?** With the graphic your team asks for every week. Let people brief the agent, review the work and ask for changes in Slack, then teach it the next thing.

**Do I need Slack?** No. Call the deployment's API, open it in LangSmith Studio, or use the terminal (`design-agent ask`). On Managed Deep Agents, long jobs that work in the background (films, parallel looks) report back in Slack; over the API, keep to requests that finish in one turn.

**Is it safe with my keys?** The sandbox never holds a credential and has no network by default. Host tools do the fetching, generating and delivering, and drop results in as files.

**Which models?** Claude Opus 5.5 by default. OpenAI and Google models, any OpenAI-compatible endpoint and the LangSmith Gateway work too. See [Models](docs/models.md).

**Can I run it on my own servers?** Yes. It's Apache 2.0, and [Self-hosting](docs/self-hosting.md) runs the same agent with Docker, an HTTP API and your own Slack app.

**Why two names?** Content Studio is what we call the agent at LangChain, and the name its Slack app gets unless you set `DESIGN_AGENT_SLACK_NAME`. The repository, package and command are `design-agent`.

**Does it replace a designer?** No. It takes the repetitive requests, so designers spend their time on the work only they can do, and their judgment lives on in its skills and checks.

**Build your own.** Fork it, add your brand tokens, fonts and skills, and deploy it to your team's Slack. [Quickstart](#quickstart). If it's useful, a star helps other teams find it.

### Built with

Deep Agents · Managed Deep Agents · HyperFrames · Paper

Also: [LangSmith](https://smith.langchain.com/) · [Chrome Headless Shell](https://developer.chrome.com/blog/chrome-headless-shell) · [FFmpeg](https://ffmpeg.org) · [D2](https://d2lang.com) · [Apache ECharts](https://echarts.apache.org) · [sharp](https://sharp.pixelplumbing.com). Logos belong to their owners. HyperFrames and Paper are partner projects we build on.

[Apache 2.0](LICENSE) · [Notices](NOTICE.md) · [Changelog](CHANGELOG.md) · [Security](SECURITY.md) · [Code of conduct](CODE_OF_CONDUCT.md)

---

## 仓库结构（GitHub Tree API，剪藏时）

顶层：`.agents/` `.claude/` `.github/` `AGENTS.md` `CLAUDE.md` `CHANGELOG.md` `CONTRIBUTING.md` `OPERATIONS.md` `SECURITY.md` `LICENSE` `NOTICE.md` `Makefile` `agent.py` `identity.py` `memory.py` `channels/` `config/` `deploy/` `docs/` `examples/` `fonts/` `install.sh` `instructions.md` `licenses/` `llms.txt` `pyproject.toml` `uv.lock` `sandbox/` `schedules/` `scripts/` `skill-binaries/` `skills-lock.json` `skills/` `src/` `tests/` `tools/` `workspace/`

**`skills/`（运行时 skills，35 个）**：brand-design · content-brief · data-viz · design-wiki · embedded-captions · faceless-explainer · film-breakdown · general-video · gif-edit · hyperframes-animation · hyperframes-audio · hyperframes-cli · hyperframes-core · hyperframes-creative · hyperframes-keyframes · hyperframes-registry · hyperframes · image-generation · image-ops · langsmith-trace · launch-film · media-production · media-use · motion-direction · motion-graphics · music-to-video · paper-design · pr-to-video · reference-study · remotion-to-hyperframes · slideshow · sound · stills · storyboard · thumbnails · visual-verify

**`.agents/skills/`（给编码 Agent 用的工作流 skills）**：brand-onboarding · deep-agents-core · deep-agents-memory · deep-agents-orchestration · design-agent-customize · design-agent-deploy · design-agent-evals · design-agent-onboarding · design-agent-troubleshoot · design-agent-update · langsmith-dataset · langsmith-evaluator · managed-deep-agents · release-check · vendor-sync

**`docs/`**：README.md · architecture/{README, context-and-failures, orchestration, surfaces}.md · cli.md · customization.md · deploy-mda.md · evals.md · integrations.md · models.md · partners/{hyperframes, paper, paper-hosted-worker}.md · self-hosting.md · sources/official-links.md · spec.md · upstream.md

---

## 补充文档 1：docs/architecture/README.md

# Architecture

One agent assembly (`src/design_agent/assembly.py`) serves Managed Deep Agents, the CLI, the self-hosted API and Slack. Runtime profiles (`src/design_agent/runtime/`) supply what differs: the sandbox backend, how files are delivered, how the lead is woken, where state persists.

| Page | Covers |
| --- | --- |
| [Orchestration](orchestration.md) | background helpers, wake-ups and check-ins, restarts and durable transcripts, pipelines, looks, render watchers, learning |
| [Context and failures](context-and-failures.md) | what the model sees, tool disclosure, cache discipline, cost, failure boundaries |
| [Surfaces](surfaces.md) | Slack progress, footer and steering on MDA and self-hosted, the API, the CLI |

Request path: a message arrives (Slack, API, CLI) → the lead's turn: the workspace is staged into the thread's sandbox (skills' scripts, gates, brand pack, fonts) → the lead plans, composes, renders in the sandbox, looks at its work, runs the gates → finals are delivered with `upload_files` and recorded in the project's `DELIVERY.json` so nothing is sent twice. Long work runs as background helpers in the same sandbox; their ends wake the lead.

Trust boundaries: model output, user files, Slack events, canvas content, MCP metadata and web pages are data. Keys live in host tools; the sandbox has no credentials and no network by default.

### The lead and its helpers

- Each thread gets one lead. It writes a plan in the thread's project folder and decides what to delegate.
- Helpers run in the background on the same sandbox and files. Each one has a heartbeat, a durable transcript, and a lead that can steer it.
- Nobody sits and waits. Render watchers and check-ins wake the lead when work lands, fails or stalls: one-time schedules on Managed Deep Agents, an in-process waker everywhere else.
- Before anything ships, the lead and helpers look at their own renders with a vision `look` tool.

### The context

- The always-on prompt is small, about 4k tokens. Workflows live in [skills](../../skills/) (`skills/<name>/SKILL.md`) and load on demand. On Managed Deep Agents they mount read-only from Context Hub.
- Brand facts come only from the brand pack. It's staged into the sandbox with its fonts and logos.
- Context stays bounded:
  - Long tool results go to files, with a short summary in the conversation.
  - Provider tool search loads MCP tools only when they're needed.
  - Helpers can clear old tool results server-side.
  - Prompt caching keeps the lead's prefix warm for an hour.

### The designer's computer

- One recipe, [`sandbox/setup.sh`](../../sandbox/), pins Chrome Headless Shell, FFmpeg, HyperFrames, sharp, ECharts, D2 and open fonts by version and sha256. It ends with a probe that renders a page, a film and a diagram. The same recipe builds:
  - the local and self-hosted Docker image;
  - the LangSmith snapshot that Managed Deep Agents clones for each thread;
  - the CI image.
- Every thread gets its own sandbox, and its files persist in `/workspace/projects/` between turns.
  - Locally, it's a container with no network, dropped capabilities, an unprivileged user, and limits capped to your machine.
  - On Managed Deep Agents, it's a LangSmith sandbox (16 GiB disk, 8 vCPU) with default-deny egress.
- Before a run, a manifest with a sha256 checklist stages skills, the checks, your brand pack and its fonts into `/workspace`, read-only.

### The checks

The checks ([`workspace/tools/`](../../workspace/tools/README.md)) run before anything is delivered:

| Check | What it catches |
| --- | --- |
| `brand-lint` | off-brand colours, fonts, radii and copy |
| `video-gate`, `hyperframes check` | non-deterministic animation, layout, motion and contrast problems |
| `artifact-contract` | a wrong size or duration, a missing file, a blank first frame |
| font check | a brand font that silently fell back, web fonts included |
| chart and graph contracts | truncated baselines, mixed units, shares that don't add up to 100%, a diagram that changed shape |

### Keys stay outside

The sandbox never holds a credential. Host tools run next to the agent, keep the keys, and drop their results in as files:

- image generation;
- voice, music and sound effects;
- transcription;
- an asset fetcher with SSRF protection;
- reading this deployment's own LangSmith traces (`list_traces`, `read_trace`);
- Paper, scoped by a policy;
- your MCP servers;
- delivery: to Slack, to `workspace/out/` locally, or attached to the LangSmith trace for API runs.

### Where each piece runs

| | Managed Deep Agents | Self-hosted |
| --- | --- | --- |
| Agent | `agent.py` (`define_deep_agent`), `identity.py`, `memory.py`, `tools/mcp.py` | the same assembly in a FastAPI app |
| Threads and memory | LangSmith; team and per-person memory layers (`memory.py`) | Postgres checkpointer and store; no memory layers |
| Sandboxes | LangSmith sandboxes from your snapshot (`sandbox/__init__.py`) | containers on your Docker host |
| Slack | managed app (`channels/slack.py`) | your Bolt app, Socket Mode or signed HTTP |
| Wake-ups and housekeeping | runtime schedules; `schedules/housekeeping.py` every 15 minutes | in-process waker and maintenance loop |
| Skills | synced to Context Hub; binary files ship as deployment files | read from the checkout |

### Models

Claude Opus 5.5 is the default, and any tool-calling model with vision works. A registry keyed by exact model id sets effort, tool search and output limits per role: lead, helper, look, tool selector and learning. The optional [LangSmith Gateway](../models.md) uses each provider's native route, so caching and context editing keep working. Every call is metered per job, helper and role, and LangSmith traces every run.

---

## 补充文档 2：docs/architecture/orchestration.md

# Orchestration

Design Agent's lead plans, delegates and reports. Helpers do the long work (builds, reviews, renders) in the background, on the same sandbox, while the lead stays free to talk. The lead ends its turn as soon as work is running and is woken when something happens. The code that makes this work is the harness in `src/design_agent/harness/`.

The harness imports only the standard library, `langchain`, `langchain_core` and `langgraph`. It never imports MDA, Slack or a sandbox client. Hosts plug in through three small protocols in `harness/protocols.py`, so the same primitives run under Managed Deep Agents, the self-hosted runtime, a plain Deep Agents graph, and the unit tests.

| Protocol | What the host provides |
| --- | --- |
| `HelperRunner` | `start(brief, effort=…)` returns a task id at once; `run(…)` awaits a `HelperResult` |
| `SandboxExec` | `execute(command, timeout=…)` in the thread's sandbox, returning an `ExecResult` |
| `Waker` | `wake(target, message, dedupe_key=…)`, `schedule(target, at, message)`, `cancel(id)` |

### The event spine

Everything that happens in the background is one `Event` (`harness/events.py`): a kind (`task.done`, `watch.stalled`, `wakeup.due`, …), the thread it concerns, its source (`t-…` helper, `r-…` watch, `p-…` pipeline, `w-…` check-in), a label, a sentence of text and structured data. Producers publish to the in-process bus. Consumers subscribe: the inbox, the cost meter, and the store-backed `EventLog`, which keeps a thread's events for a week. Producers and consumers never import each other.

The attention rule (`harness/policy.py`) decides what each event costs the lead:

| Delivery | Events | Effect |
| --- | --- | --- |
| `NOW` | helper finished, failed or interrupted; watch failed, stalled or finished; check-in due; someone must act | wake the lead |
| `NEXT` | helper progress notes, pipeline steps | ride along with the lead's next turn |
| `LOG` | starts, per-frame render progress, usage | recorded only |

`data["urgent"]` promotes any event to `NOW`; `data["ping"]` marks one a person must act on. Unknown kinds wake the lead, because nothing is dropped silently.

### Waking the lead

`Inbox` (`harness/inbox.py`) is built once per process with the host's `Waker` injected. Work that starts on a thread registers a `WakeTarget` for it: the thread id plus an opaque `channel` the adapter uses (where replies post). `NOW` events landing within five seconds of each other, or while a wake-up is under way, are delivered together as one notice (`harness/notices.py`). The notice says how each helper ended, what is still running, and the likely next step. A lead turn that starts for any other reason (a message from the person) carries whatever is still waiting, so the lead is not woken again for news it already has.

A reply of exactly `[quiet]` means there is nothing worth saying. Locally and self-hosted, the runtime posts wake-up replies itself and drops it. On MDA every scheduled run's reply is posted by MDA's schedule delivery, an `after_agent` hook installed ahead of every authored middleware, so it runs last and cannot be skipped (and an empty reply would make it repost an earlier one). There `QuietReplyMiddleware` rewrites a final `[quiet]` into one line about the work still running ("Still working: act 2 (12 min)."). The upstream ask: let a scheduled run end without a post.

A wake-up counts as delivered only when the waker's `WakeReceipt` says the lead's run started. A failure, or a receipt with `delivered=False`, is retried after 10 s, 1 min and 5 min with whatever landed meanwhile. After the last retry the events are held for the lead's next turn. Each wake-up carries a `dedupe_key` built from the thread and the batch's event ids, so a retry of a batch that did start a run can be recognised by the adapter.

Adapters live outside the harness. Self-hosted, the waker enqueues a run in the API process with a Postgres timer table for check-ins (SPEC §7.4). On MDA it uses public one-time runtime schedules on the current thread (`runtime/waker/mda.py`), with these constraints from managed-deepagents 0.9.4:

- **Latency.** A schedule's `at` must be at least a minute ahead and is rounded up to the next whole minute, so a wake-up lands 60–120 s after the event, plus the scheduler's own lag. The inbox coalesces meanwhile, so one wake-up carries everything that happened. Measuring the lag needs a deployment; the upstream ask is a public "start a run on this thread now" API.
- **Delivery receipt.** The woken run's first hook writes the wake key to the deployment store; the waker polls for it for up to five minutes before it counts the wake-up delivered.
- **Who can schedule.** Only a run's own runtime can create a schedule, for that run's thread. The waker keeps each thread's latest `runtime.schedules`, so after a restart a thread has none until a run happens on it.
- **Restarts.** While a thread has background work, it holds one sentinel schedule 15 minutes ahead, which the process pushes forward every five minutes. It fires only once that process is gone; the sentinel turn sweeps the ledger at once, so its reply says what the restart stopped. If it fires while the work still runs (a postponement failed), the turn ends without a model call.
- **Ownership and caps.** Agent-owned same-thread schedules need a channel binding; runs without one (Studio, the API) schedule as the caller. Each owner holds at most `MDA_MAX_AGENT_SCHEDULES` (default 20) unexpired schedules; a thread uses one sentinel plus its pending wake-ups and check-ins.

### Background helpers and restarts

`harness/tasks.py` runs a helper as an asyncio task and publishes `task.started`, `task.progress`, `task.done`, `task.failed` and `task.cancelled`. `CURRENT` (and the meter's `CURRENT_TASK_ID`) names the helper whose code is running, so its tools, transcript and cost are attributed to it.

A process restart kills in-process helpers silently, so the ledger (`harness/ledger.py`) keeps a heartbeat record per helper in the deployment store, refreshed every minute. The record also holds how the helper ended until the lead has been told. `ledger.sweep` finds records whose heartbeat is more than three minutes old: a helper still marked running is published as `task.interrupted`, and one that ended but was never delivered is published as a normal finish. Check-ins the lead scheduled (`harness/wakeups.py`) live in the ledger too, so one a restart swallowed fires late rather than never.

### Durable helpers

`TranscriptCheckpointMiddleware` (`harness/durable.py`) saves a helper's history to the store as it works: tool results before each model call, the reply after it. `resume_task(id)` restarts the worker with `resume_brief(id)`, and the middleware swaps that one-line brief for the saved transcript and a closing note. What is saved is always valid to send back:

- viewed images and other media become text stubs naming their file (`[image you viewed: …]`);
- thinking blocks are dropped, since a provider may reject replayed thinking after an edit;
- a tool call a restart cut off gets a result saying its effect is unknown;
- past the byte cap (8 MB), the oldest steps after the brief are dropped, with a marker.

### Pipelines as code

`WorkflowRunner` (`harness/workflow.py`) runs registered pipelines (`fn(ctx, **args)`) in the background. Control flow is Python; every creative call is a helper brief. `ctx.agent` runs one helper and can validate the JSON its report ends with, sending it back with the problem up to `attempts` times. `ctx.parallel` and `ctx.pipeline` fan out, and `ctx.ask` parks the run until a person answers through the lead. Every step is memoized by the hash of its brief, effort, schema and label. A re-run after an answer, a restart or `resume_task("p-…")` replays finished steps instantly and runs only the rest. Pipelines must be deterministic: no clock, randomness or environment inside them.

### Parallel looks

`Looks` (`harness/branches.py`) copies a project into `looks/<name>/` per creative direction (without `looks/`, `out/` and `node_modules/`, which is linked back) and starts one helper per look. Each helper writes `look.json` with its direction, summary and stills. `LookWatch` hears every look's helper end and asks a person to pick. `pick` backs the project up to `looks/_backup-<n>/`, then merges the chosen look over it with newer files winning. Looks belong at the stills stage, never full renders.

### Render jobs and watchers

`WatchRunner` (`harness/watch.py`) polls a source every 15 s, parses new output and publishes `watch.progress` (fraction, phase, ETA), `watch.done`, `watch.error`, or `watch.stalled` after five minutes with no new output and no progress. A watch that stalled and starts moving again says so urgently. Polling the sandbox also keeps an idle sandbox alive. `SandboxDetachedRender` (`harness/offload.py`) runs a command under `nohup`/`setsid` with its output in a log and its exit code in `.exit`; the exit code is the verdict, and the log feeds progress.

Parsers come from `parser_for(kind)`:

| Kind | Reads |
| --- | --- |
| `heartbeat` (default) | any output as a sign of life; failure lines (`Error`, `Traceback`, `Killed`, OOM, `exited with code N`) end it |
| `hyperframes` | `hyperframes render` progress (below) |
| `ffmpeg` | `-progress` blocks or the stats line; a percentage with `total_frames` or `duration_seconds` |
| `json` | a status file (`status`, `step`, `progress`, `eta`), read with `SandboxJsonSource` |
| `regex` | any `N/M` counter or percent, with optional done and error patterns |

HyperFrames 0.8 writes two forms of progress. Without a TTY (a detached job's log) it prints a machine line per step beside a plain progress line:

```text
  ██████████░░░░░░░░░░░░░░░  42%  Streaming frame 18/60
@hf-progress {"code":"capture","done":18,"total":60,"pct":42}
```

Under a TTY it redraws one colored bar with `\r` and prints no machine lines. The stage codes are `compile`, `extract_video`, `process_audio`, `start_browsers`, `capture`, `encode` and `assemble`. The parser maps either form to one phase name, takes `pct` as the fraction, and reads frame counts during capture and encode. A render is done at `100% Render complete`, and the parser waits one read for the `◇ ` line that follows. It fails at a `NN% Failed: …` progress line, `Render cancelled`, the CLI's error box (`✗ Render failed`, `✗ Not a directory: …`) or a killed process. It deliberately ignores generic `Error` text, because a best-effort render logs the page's own script errors as warnings and still finishes. Real captured logs live in `tests/unit/harness/fixtures/`.

`WatchLedger` keeps each watch's record in the store: what it reads and how, where it stands, and the host's wake data. The record is refreshed while the watch runs and kept after it ends until the lead has heard the end. After a restart, `sweep` publishes a lost end again, or reports a cut-off watch as `task.interrupted`. That record stays resumable for 24 hours, so `resume_task("r-…")` can watch the same log (and exit file) again under the same id.

### Context and cost

- `harness/cache.py` rewrites every `cache_control` breakpoint in a request to one TTL (1h for the lead, which sleeps between turns; 5m for helpers) and adds a breakpoint on the tools array when the tools are Anthropic-shaped. A payload for another provider passes through unchanged. `StaggerGate` holds a wave of sibling helpers until the first response starts, so they read the shared prefix rather than each writing it. `PrefixWatch` logs which part of a prefix changed.
- `harness/context.py` builds Anthropic's server-side tool-result clearing for helpers (opt-in) and counts image tokens by their real cost instead of LangChain's flat 85 tokens, reading image sizes from file headers with the standard library.
- `harness/meter.py` prices every call from its uncached, cache-write (by TTL), cache-read and output tokens, and sums a job (thread) per source and per role. Prices live in `config/prices.toml`, one entry per provider and model with the date it was checked and its public source. Long-context tiers are included where a provider has them. Every figure is an estimate; the provider's invoice is the source of truth.

### Learning

`ObservationStore` (`harness/learning.py`) keeps per-skill observations: what a past job showed, with evidence. A near-duplicate (token Jaccard ≥ 0.8 or difflib ratio ≥ 0.85) merges into the existing note and counts again. Ratings add helpful or harmful votes. A note whose harmful votes lead by two is dropped by `prune`. Notes are `proposed` until seen again, rated helpful or given by a person. The rendered list is headed as untrusted notes the next job weighs, never rules.

### Upstream status

| Primitive | Upstream today | Plan |
| --- | --- | --- |
| Background helpers on the lead's sandbox | Deep Agents async subagents run as server runs on their own threads; MDA does not build them yet | migrate when MDA runs async subagents on the lead's sandbox, then delete the in-process runner |
| Durable transcripts | server runs are checkpointed; in-process subagents are not | keep the stub rules if replay still needs them |
| Wake-ups | Deep Agents notifies a parent when a same-deployment async subagent finishes | becomes the native path for helper results once async subagents ship in MDA |
| Event bus, attention rule, watchers, detached jobs, pipelines, looks, meter | none | candidates to upstream as they are |

---

## 补充文档 3：AGENTS.md

# Working in this repository

Design Agent is an open-source design agent with one shared core and two deployment paths: Managed Deep Agents (MDA, recommended, one command) and self-hosting (Docker). It makes on-brand stills, decks, diagrams, charts and motion graphics from a chat thread, on its own computer, with HyperFrames as the video engine and Paper as the shared canvas. Keep company-specific brands, credentials, fonts, reference films and operational history out of the public repository.

### What people ask you to do here

Most people open this repository to set up, brand, deploy or extend their own design agent. Each job has a skill in `.agents/skills/` (Claude Code also finds them through `.claude/skills`). Open the matching `SKILL.md` and follow it:

| The person wants to… | Skill |
| --- | --- |
| set it up, see the demo, connect a model or Paper | design-agent-onboarding |
| add their brand from a guide, website, files or a Paper file | brand-onboarding |
| put it in their team's Slack: deploy to MDA or self-host | design-agent-deploy |
| teach it a new deliverable, rule or tool, or change models | design-agent-customize |
| check whether a change made it better or worse | design-agent-evals |
| fix something that fails | design-agent-troubleshoot |
| update their fork to the latest version | design-agent-update |
| move vendored skills forward (maintainers) | vendor-sync |
| cut a release (maintainers) | release-check |

The harness under the agent has its own skills, vendored unmodified from LangChain's public packs:

| To work on… | Skill |
| --- | --- |
| the agent loop, middleware, skills format and configuration | deep-agents-core |
| backends, stores and persistence | deep-agents-memory |
| subagents, planning and human approval | deep-agents-orchestration |
| the Managed Deep Agents project and deploy | managed-deep-agents |
| reading traces of a run that went wrong | langsmith-trace |
| turning traces into evaluation datasets | langsmith-dataset |
| LangSmith evaluators and experiments | langsmith-evaluator |

They are written for any project, so where they disagree with this file, this file wins.

For anything else, start from the map of every document, docs/README.md. Never print a key, and ask before deploying, connecting accounts or spending money. The rest of this file is the contract for changing the agent itself.

Code and tests are the source of truth; docs/spec.md records the product intent behind them.

### Read first (to change the agent)

1. docs/architecture/: how the assembly, harness, surfaces and sandbox fit together.
2. docs/spec.md: product behavior, slices and acceptance criteria.
3. docs/sources/official-links.md: primary documentation for every external contract; docs/upstream.md: seams we work around.

### Folder boundaries

- `.agents/skills/`: workflows for coding agents (onboarding, brand setup, vendor sync, release), plus LangChain's Deep Agents and LangSmith skills, vendored. `langsmith-trace` links to its runtime copy in `skills/`. `.claude/skills` links here. Never synced to the runtime sandbox.
- `skills/<name>/`: runtime skills, ours and vendored, exactly one level deep (MDA discovers only `skills/<name>/SKILL.md`, does not follow symlinks inside `skills/`, and evals do not see a symlinked skills root). MDA mounts them read-only at `/skills/` from Context Hub, outside the sandbox filesystem; the workspace stager copies their scripts and assets into `/workspace/skills/<name>/` so they can run. Context Hub takes only UTF-8 text, so a skill's binary files live in `skill-binaries/<name>/` and the stager merges them back; a plain `mda deploy` fails on one binary file under `skills/`.
- Vendored skills keep their upstream names (HyperFrames and `langsmith-trace` under `skills/`, LangChain's coding-agent skills under `.agents/skills/`) and are listed in `skills-lock.json` (each upstream's repo, commit and license, per-file sha256). Never patch vendored files; override in a sibling skill or summary of ours, or upstream the change. CI fails on drift.
- `workspace/brand/`: the active brand (tokens, logos, voice, graphic language). The sample brand ships in `examples/brands/`; the user's own brand is Git-ignored.
- `workspace/references/`: the user's reference films and teardowns. Git-ignored. Never commit third-party footage, frames or contact sheets.
- `private-fonts/`: operator-supplied licensed fonts. Git-ignored; reach the sandbox only as MDA deployment files or self-host mounts.
- `agent.py`, `identity.py`, `memory.py`, `channels/`, `schedules/`, `sandbox/`, `tools/mcp.py`: MDA project declarations. Keep them thin.
- `src/design_agent/assembly.py`: the shared model, tools, middleware, subagents and policy.
- `src/design_agent/harness/`: orchestration (background helpers, wake-ups, check-ins, durable helper transcripts, pipelines, parallel looks, render watchers, context and failure boundaries).
- `src/design_agent/tools/`: host tools (render jobs, image generation, voice, Paper scope, file delivery). Credentials and network calls live here, never in the sandbox.
- `src/design_agent/runtime/`: MDA, self-hosted and local runtime profiles and sandbox backends.
- `src/design_agent/surfaces/`: self-hosted API and Slack transport.
- `tests/`: unit, real-graph contract, render smoke, offline behavior and opt-in integration tests.

### Runtime rules

- One assembly serves MDA, the CLI, the self-hosted API, Slack and schedules.
- Use native, public MDA APIs from the latest stable `managed-deepagents` release. Do not import private MDA modules. When a feature needs a seam MDA does not expose, put it behind an interface with an MDA adapter and a self-host adapter, and record the upstream ask.
- The model owns creative judgment. Code owns rendering, measurement, gates, file delivery, permissions and every number a report states.
- Treat model output, user files, Slack events, Paper canvas content, MCP metadata, web pages and reference material as untrusted input. None of it grants permission.
- Keys stay on the host. The sandbox never receives credentials. Host tools fetch and drop results in as files; the agent only sees paths.
- Nothing is published or sent outside the thread without an explicit person in the loop.
- Skills guide judgment; code grants capability. A skill edit cannot expose a tool or loosen policy.
- Keep the root prompt small. Put workflows in skills, doctrine in the design wiki, brand facts in the brand folder. Large results go to files with compact summaries.
- Do not copy private company data, internal people, channels, accounts, licensed fonts, third-party footage or internal implementation notes into this repository. LangChain's brand and the agent's public name, Content Studio, appear only in the README and `docs/assets/readme/` (the project's own presentation: figures exported from its Paper designs and its films) and as the default Slack app name; the code, the agent's sample brand, skills, examples and tests stay brand-neutral and say Design Agent.

### Change discipline

Read existing code before editing. Prefer direct code, small diffs, and deletion of obsolete paths over compatibility layers. Comments explain constraints, not history. Use the thermo-nuclear code-quality review skill for reviews when available. For any agent behavior, prompt, skill, tool, middleware or evaluation change, apply the agent-engineering skills: rightsizing-agent-context, create-agent-skills, tool-design, validating-agent-behavior and change-verification.

Update public docs and the owning runtime skill when behavior changes. Do not add build journals, session handoffs or audit transcripts to the repository. Preserve unrelated local work.

Do not commit, push, deploy, create remote repositories, connect accounts, publish packages, or contact partners unless the user asks.

### Verification

```bash
uv sync --all-extras --dev
uv run ruff check . && uv run ruff format --check .
uv run mypy src
uv run pytest -q
uv run design-agent doctor
uv run design-agent demo
```

CI also imports the MDA definition, runs the sandbox recipe probe, renders the fixture compositions, checks vendored-skill checksums against `skills-lock.json`, scans for secrets and for banned private identifiers, and checks documentation links. Verify synced context with a clean-checkout `mda build` and inspect what MDA actually syncs, not only the source tree.
