---
url: "https://github.com/NeuveAI/neuve"
clipped: 2026-11-18
title: "NeuveAI/neuve — Large code changes never felt so small（开源代码评审工具）"
---

# 仓库元信息（GitHub API，剪藏时）

- 仓库：NeuveAI/neuve（Organization: NeuveAI）· 描述："Large code changes never felt so small"
- 主页：https://neuve.ai · 主语言：TypeScript · 许可：MIT
- 创建：2026-11-16 · 最近 push：2026-11-17 · 默认分支：main
- Stars 3 · Forks 0 · Open issues 0 · 仓库体积约 19 MB

---

# README.md（原文）

# Neuve

[![License: MIT](https://img.shields.io/badge/license-MIT-1C1A16)](LICENSE)

> Hero 图 alt：Large code changes never felt so small - Focus only on parts that require human attention or approval, with the certainty your codebase deserves.

> [!NOTE]
> Neuve is early. Expect rough edges and changing commands.

Neuve is an open-source code review tool that allows developers and agents ship code with more confidence.

Neuve works using a hybrid agentic, graph and semantic layer on your code changes to abstract changes into different lanes.

**Automation**: CI/CD covered code with low complexity.
**Agentic**: Code that can easily be validated or its complexity does not require human attention or validation.
**Human Suggested**: Code that have risen above of changes that could safely be assigned to agentic reviews, but does not warrant immediate human attention.
**Human Required**: Code that either is difficult to validate through agentic checks, has high complexity score or has enough context signals that requires human attention.

Today it runs on your machine, using your Codex or Claude Code local setup. Neuve posts no reviews or comments to GitHub at this moment.

Neuve works by creating a review desk with changes assigned to one of four lanes as review units. Each range given to Neuve might come with more than one review desk, which is called a batch.

## Why

I started out building agents for QA, and soon realized that the highest friction point wasn't on QA but much earlier in the SDLC. From scoping work, to ensuring the right work is respecting the scope planend and afterwards auditing and reviewing that work to a satisfactory status.

As software engineers we used to say _done is better than perfect_ as a phrase to promote speed and avoid drawn out iterations. We now can get to **done** very easily and quickly, but getting to **good** is a challenge for mosts teams trying to operate at agentic speeds and volume.

I wanted large changes to stop being something to be overwhelmed by or hard to process within teams, so decided to rethink reviews all together.

## Quick start

One line installs the `neuve` command on Macs with Apple silicon (still working on Linux and Win builds).

```sh
curl -fsSL https://neuve.ai/install.sh | sh
```

Reviews run on your own model account. Choose one of three:

- **An Anthropic API key**, the default. Put it in the shell you run `neuve review` in:

  ```sh
  export ANTHROPIC_API_KEY=...
  ```

- **A Claude subscription**, through Claude Code. Install Claude Code, then sign in with its own login if you haven't yet. Neuve runs reviews through Claude Code and never reads your sign-in. Choose it with `neuve setup --route claude`.

  ```sh
  claude auth login
  ```

  If you installed Neuve before it offered this choice, update it first: run `neuve upgrade`, or the install line again if your `neuve` has no `upgrade`. An older `neuve` refuses a settings file that names `claude`.

- **A ChatGPT subscription.** Choose it with `neuve setup --route openai`, then sign in:

  ```sh
  neuve auth login openai
  ```

Run `neuve setup` in your repository. At a terminal it asks which of the three to use; `--route claude` or `--route openai` chooses without asking. It ends by checking that your sign-in is available. `neuve doctor` names anything that's missing and how to fix it. For a Claude subscription, doctor checks only that Claude Code is installed; `neuve auth status claude --check` checks the sign-in once Claude Code is installed, without spending usage.

```sh
cd /path/to/your/repository
neuve setup
```

In the same repository, point Neuve at a commit range, then open the review in the review desk:

```sh
neuve review main..my-branch
neuve review desk
```

`neuve review` starts a review and follows its run. A background worker runs the review, so closing the terminal doesn't stop it. Without `neuve setup`, `neuve review` and `neuve review desk` stop and print the setup command to run. `neuve review desk` prints the desk's address, such as `http://127.0.0.1:4141/`. The desk stays open until you press Ctrl-C. While it's open, `neuve review` also prints a link to the review. The [CLI guide](apps/neuve-cli/SETUP.md) covers setup, review settings, the desk and the other commands, and `neuve --help` lists them all.

### Upgrade and uninstall

```sh
neuve upgrade
curl -fsSL https://neuve.ai/install.sh | sh -s -- --uninstall
```

`neuve upgrade` installs the latest release. To install a given release, add its tag, such as `neuve upgrade --version v0.1.0`. The second line uninstalls Neuve.

### If an install was cut off

If you close the terminal or press Ctrl-C during an install, upgrade or uninstall, the installer puts the previous version back. Run the same command again.

If the machine lost power or the install was stopped with `kill -9`, the next run may stop with "Another install holds …/.install.lock". The folder is in your install folder, `~/.neuve` by default. If none is running, delete that empty folder. Then run the same command again, adding `--force` after `sh -s --`:

```sh
rmdir ~/.neuve/.install.lock
curl -fsSL https://neuve.ai/install.sh | sh -s -- --force
curl -fsSL https://neuve.ai/install.sh | sh -s -- --uninstall --force
```

The second line is for an install or upgrade, and the third for an uninstall. After an upgrade, use the install line above, because `neuve upgrade` has no `--force`.

If you installed into another folder with `NEUVE_INSTALL_DIR`, delete `.install.lock` in that folder instead. The installer's message gives the lock's full path, so you can copy it from there. Keep `NEUVE_INSTALL_DIR` set when you run the command again. Without it, the installer uses `~/.neuve`.

`--force` also lets the installer replace a copy built from source. Use it only when no other install is running.

Each user account has one launchd agent on macOS, or one systemd timer on Linux, that starts background workers. If you install into two folders, the install that finishes last takes it over. Running two installs into different folders at the same time isn't supported.

### Working on Neuve itself

To build Neuve from source or work from a checkout, see [CONTRIBUTING.md](CONTRIBUTING.md).

## How it works

```
commit range ─▶ goal, batches and parts ─▶ four agents look for problems ─▶ each finding checked again ─▶ you record your review in the review desk
```

| Step                    | What happens                                                                                                                                                                                                                                                                                                    |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Goal, batches and parts | Neuve states the change's goal and what done means, from the ticket or plan it can fetch, and says when there isn't one. Each changed line belongs to one part, even when a run is cut off                                                                                                                      |
| Looking for problems    | Scope fit says whether the change meets what done means, or that a person must judge it. Correctness follows the changed code to a failure you could see. Risky edges checks data, auth and interfaces such as exports, config and wire formats. Tests checks whether tests or other proof show what done means |
| Checking                | The check runs against evidence that Neuve's code gathers, with no lookup tools for the model. A finding that couldn't be checked is counted, never shown as a comment                                                                                                                                          |
| Your call               | From the overview of a batch, open one of its parts. The checked findings sit beside the lines they're about, and you record your review in the review desk                                                                                                                                                     |

The [product vocabulary](vocabulary.md) defines Neuve's terms, including the review unit, its name for a part, and the [architecture overview](architecture/README.md) describes the six layers behind them: harness, system of records, graph, CLI, MCP and web app.

## Documentation

- [Contributing](CONTRIBUTING.md): building Neuve from a checkout, the contributor check and how review works
- [CLI guide](apps/neuve-cli/SETUP.md): setting Neuve up in a repository, review settings, the desk and the other `neuve` commands
- [Architecture](architecture/README.md) and its records: how the system is built, and why
- [Packages](CONTRIBUTING.md#packages): the repository's packages and what each one is
- [Product vocabulary](vocabulary.md) and [foundations](foundations): what Neuve's terms and ideas mean
- [Design decisions](process/decisions.md): Neuve's product decisions, which override older text

## Contributing

Neuve is built by coding agents under independent review, and contributions go through the same review. [Contributing](CONTRIBUTING.md) explains how to propose a change, how review works and what a pull request needs. Please read the [code of conduct](CODE_OF_CONDUCT.md), and report security problems privately as the [security policy](SECURITY.md) describes.

## Credits

Neuve's host and channel packages (`@ace/host` and `@ace/channel`) are adapted from GitHub Next's [Ace](https://github.com/githubnext/ace), and Neuve Graph is a fork of [Entire Graph](https://github.com/entireio/entire-graph). Both are MIT-licensed. The review desk's diffs render with a modified copy of Pierre's [`@pierre/diffs`](https://github.com/pierrecomputer/pierre) (Apache-2.0). The notices for all three are kept in the repository.

Other third-party components, icons and fonts keep their own licenses. Some packages list them in a notice, such as the web app's interface package ([`apps/neuve-web/packages/ui`](apps/neuve-web/packages/ui/NOTICE.md)).

## License

[MIT License](LICENSE) · Copyright (c) 2026 Neuve AI

---

# 附录：architecture/README.md

> 来源：https://github.com/NeuveAI/neuve/blob/main/architecture/README.md（README 引用的子文档，原文）

# Architecture

This folder describes the system that follows from the owner's [design decisions](../process/decisions.md), recorded on 16 October 2026. The [foundations](../foundations/) define what the product means, and the [vocabulary](../vocabulary.md) defines its terms. This folder covers how the system is built: its layers, how they communicate, where data lives, and why. Each file in [`decisions/`](decisions/) records one architectural decision and links back to the owner decisions it comes from.

These records describe the target architecture. Much of the current code does not match it yet. The correction milestones close that gap.

## Layers

Neuve has six base layers. Each owns its own internal abstractions and talks to the others only through the boundaries listed here. See [0001](decisions/0001-layers-and-responsibilities.md).

| Layer | Owns | Does not own |
| --- | --- | --- |
| **Harness** | The final-review pipeline, from scope gathering to dispositions: hybrid retrieval, refinement and reranking, deterministic checks, matching units to context, digests, the lane matrix, pre-generated dispositions and review-delivery content. It runs as a client of the system of records. | Durable state, identity, and the UI |
| **System of records** | A library that every process links, over one SQLite file in `.git/neuve/` (8.3, 8.6). The append-only record of what happened: batches, units, context references, assessments, dispositions, comments, cycles, and agent identity and activity. It also holds the content-addressed computation store and the event queue that drives background work. | Model calls, retrieval logic, and agent orchestration |
| **Graph** | The semantic layer: indexing code and local docs and keeping their embeddings, with the provenance metadata needed for citation and refinement. It is updated incrementally as commits land, and served by one shared instance per machine ([0013](decisions/0013-graph-search-service.md)). | Being the only structural analyser. Structural and static analysis are separate, explicit operations ([0006](decisions/0006-hybrid-retrieval-and-context-references.md)). |
| **CLI** | The agent-facing interface for inner-loop and outer agentic reviewers: recording work, evidence and identity, and drilling into review data. It also handles install, hooks and the local-development command. | Business rules that belong to Records or the harness |
| **MCP** | Exposing Neuve's tools to coding agents over stdio, for the lifetime of the agent's session | Long-lived ports or shared daemons |
| **Web app** | An event-driven interface for humans: the batch overview and review desk, projected from Records, with commands relayed back to it | Business logic. The BFF only projects and relays. |

ACP connects Neuve to external coding harnesses such as Codex and Claude Code when they are available. It is a connection, not a layer, and final review never depends on it.

## How work flows

| Step | What happens | Where |
| --- | --- | --- |
| 1. Capture | Git hooks and agent hooks call the CLI. It writes the event, the agent's fingerprint and the commit to a spool, then returns. The worker ingests the spool. | CLI → spool → Records |
| 2. Queue | Records turns events into background jobs: embeddings for new blobs, graph updates, and pipeline runs for review requests. | Records |
| 3. Work | A worker starts when jobs are queued and exits when idle. A launchd agent from the release install makes sure queued work is picked up. | Worker (harness and graph) |
| 4. Reuse | Every computation is looked up by its input hash before it runs. Ranges are immutable, so finished work is never redone. A partial hit reuses what exists and reranks the rest. | Computation store |
| 5. Review | The harness writes the batch package. The web app shows it to humans, and the CLI shows it to agents. Dispositions are appended as records. | Records → web app and CLI |
| 6. Cycle | When every needed disposition is in, the cycle ends. Comments, including those on approvals, become context for the next cycle. | Records |

See [0002](decisions/0002-event-driven-processes.md) for the process model and [0004](decisions/0004-persistence-and-range-immutability.md) for storage.

## Decision index

| # | Decision |
| --- | --- |
| [0001](decisions/0001-layers-and-responsibilities.md) | Six layers with their own abstractions. Records stores and coordinates; the harness computes. |
| [0002](decisions/0002-event-driven-processes.md) | Event-driven. Hooks capture quickly, jobs queue, and a worker runs on demand with a launchd backstop. No always-on service. |
| [0003](decisions/0003-effect-native-harness-runtime.md) | The incumbent Effect runtime and its validation. Superseded for agent execution by 0014. |
| [0004](decisions/0004-persistence-and-range-immutability.md) | An append-only record log plus a content-addressed computation store. Ranges are immutable, nothing is recomputed, and full session data is kept for 30 days. |
| [0005](decisions/0005-harness-pipeline.md)                        | The fixed review-shape trial: Frame, Map, four finders, Verify, code fill, Link and Publish, with bounded work and full range coverage.                                                       |
| [0006](decisions/0006-hybrid-retrieval-and-context-references.md) | The semantic layer, then procedural enrichment, then agentic refinement, with one context-reference contract feeding the context margin. |
| [0007](decisions/0007-batches-units-lanes-and-cycles.md)          | The established batch and lane policy, with the fixed-shape trial's proximity units, unassessed coverage and defect-only dispositions.                                                        |
| [0008](decisions/0008-model-routes-and-budgets.md) | Anthropic API and ChatGPT subscription routes freeze accounts and budgets while preserving legacy keys. Run configurations admit the Claude subscription route. |
| [0009](decisions/0009-identity-without-auth.md) | No auth. Agents are identified by fingerprint and alias, and git credentials are enough. |
| [0010](decisions/0010-release-and-distribution.md) | A binary distribution installed on the PATH. Tests never touch the installed binary, and end-to-end runs test the release. |
| [0011](decisions/0011-zod-schemas.md) | Zod is the one schema library, and Effect consumes it. |
| [0012](decisions/0012-stack-store.md) | A third kind of persistence: a user-level stack store of third-party sources and their indexes, outside Records, rebuildable and evicted. |
| [0013](decisions/0013-graph-search-service.md) | Graph search runs in one shared instance per machine that schedules keyed units, prepares few bases and bounds its cache; the harness is its client and never waits on Graph without a bound. |
| [0014](decisions/0014-ace-style-pi-harness.md)                    | Ace-style tasks within Effect and Bun, with pi conversations and isolated Claude Code loops behind the runtime contract. Claude stages are enabled.                                         |
| [0015](decisions/0015-range-snapshot-and-git-reader.md) | A review reads a range snapshot that the Git reader makes; hunk text comes from Git rather than durable records. Accepted as the reference; not built. |
| [0016](decisions/0016-shared-work.md) | Work more than one caller can ask for runs once per idempotency key, and every other caller subscribes to its response. The only design for it; not built across processes. |
| [0017](decisions/0017-runtime-contract-and-accounts.md) | Named accounts and isolated runtime loops, with shared tools and stage policy. Claude stages are enabled; account removal and file-watch gaps stay explicit. |

## Keeping these records true

When an owner decision changes, update the matching record in the same change and delete superseded text, which git keeps. A record describes the target until the code reaches it. Code that disagrees with a record is either a defect or a sign that the record needs the owner's revision; it never silently redefines the architecture.

---

# 附录：vocabulary.md

> 来源：https://github.com/NeuveAI/neuve/blob/main/vocabulary.md（README 引用的子文档，原文）

# Product vocabulary

Working vocabulary, updated 2026-10-16. These definitions consolidate user-established concepts. Numbers such as 4.11 refer to entries in the owner's [design decisions](process/decisions.md). Linked foundation documents hold the detailed rules and remaining questions. Algorithms and data structures are not selected by these names.

## Review units and context

The [review-unit foundation](foundations/04-review-units-and-context.md) defines grouping, decision scope, and reclustering.

| Term                                  | Meaning                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Hunk                                  | A Git diff hunk, the closest base unit for Neuve's review model.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Review unit or cluster | Synonyms for a grouping of changed line ranges by responsibility, representing work to be reviewed in a particular cycle; a large hunk can be split across units, and within one lane a file's changes are one unit. A unit has an agentic or human responsibility segment. Supporting evidence is its surrounding context; neighboring hunks included for context retain their own responsibility and disposition. Attribute the selected context to the unit and cycle. Grouping can change between cycles; continuity across patches still needs a precise definition. A chunk is not a review unit; see chunk under retrieval and investigation. |
| Committed graph                       | The durable graph built over time from committed repository state and history. Repository setup ingests the existing commit history and current committed state. Later commits extend it through ongoing ingestion. Its nodes and relationships can carry commit, blob, historical, and prior-review attribution.                                                                                                                                                                                                                                                                  |
| Current review batch                  | The work reviewed together with a single focus. A review takes a commit range attributed to a base branch as input; the range is its fingerprint, because Git immutability means the same range always carries the same changes (1.4). The harness splits one range into one or more batches, each covering one behaviour or obligation (4.1, 4.10). Within a batch, review units group changed line ranges by responsibility; within one lane, a file's changes are one unit (10.120). A commit creates durable identity and lineage, but does not confer approval; only a recorded disposition does. |
| Transient context                     | Same-batch context: a contextual marker drawn from other changed work inside the commit range selected for review. The selected range can vary: a reviewer may narrow it to commits ahead of the base or look at commits outside it. Work outside the selected range is not same-batch context; it is context that carries its own review status, such as reviewed in an earlier cycle or not yet reviewed (4.12). It can help assess units of the same batch or of other batches from the same range, but it must be visibly identified as unapproved same-batch context. It has no prior approval, cannot transfer a disposition between units, and cannot appear as durable committed-graph evidence. |
| Decision scope or surrounding context | Synonyms for the supporting information used to assess a review unit in a given cycle, including code connected through dependencies, references or blast radius, Git history, tickets, documentation and prior review information. Contextual markers retain traceable source fragments and appear through the Context margin. This describes what informs the decision, separately from the work receiving a disposition. |
| Reclustering                          | Recomputing review units for a new cycle around the work needing decisions, while retaining useful surrounding context and prior review history.                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Decision signal                       | An indication that work requires a review decision in the current cycle. A changed outcome or side effect can create this signal even when the affected code's text is unchanged.                                                                                                                                                                                                                                                                                                                                                                                                  |
| Contextual marker                     | A retrieved piece of semantic context that corroborates an attribution to a particular hunk. It is smaller than its source, such as a ticket, Git history entry, dependency graph, or document, and remains traceable to that source. A marker can support attribution to multiple review units through shared dependencies or blast radius. Markers are retrieved and linked per hunk; risk, complexity, and classification are assessed for the review unit as a whole using its surrounding context. It is not the whole source or the resulting assessment or lane assignment. |
| Contextual signal                     | Information that helps explain the work without itself demanding a review decision. Feedback on that information can still lead to later changes.                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Read-only neighbor                    | A neighboring hunk included in a review unit or cluster that belongs to a different lane from that cluster's agentic or human responsibility segment. It supplies context while retaining its own lane, decision authority, and review attribution. It can be commented on, but does not receive the cluster's decision.                                                                                                                                                                                                                                                           |
| Isolated briefing                     | A generated, advisory briefing that presents context beyond a single review unit, including shared dependencies and verification across separate units or supported uncertainty about broader effects. It does not require a disposition or replace the dispositions of the related units.                                                                                                                                                                                                                                                                                         |
| Contextual briefing                   | A generated briefing of relevant context and historical information, with connected sources for attribution and drilldown. Its purpose is to reduce the reviewer's tax of reconstructing that information while keeping the supporting detail accessible. Exact UI presentation remains open. Its descriptive phrases link to the part of the change they describe. |
| Digest                                | A generated briefing at one of two scopes (4.8). The **batch digest** is the isolated briefing on the batch overview. The **unit digest** is the contextual briefing on the review desk, with phrase links and citation chips; its context is narrower, one unit within its batch. Digests are advisory: they need no disposition and never confer approval. |

### Cycle attribution and the prepared package

Reviewed work and the context supplied to assess it have separate attribution. The [cycle attribution rules](foundations/02-review-modes-and-cycles.md#cycle-attribution-and-immutable-preparation-snapshots) describe the cycle boundary and keep product meaning separate from implementation.

| Term or relationship | Meaning |
| --- | --- |
| Review-unit context attribution | Selected supporting context belongs to the exact review unit and the cycle in which it informed review. Reusing a source in a later cycle retains its historical attribution and requires explicit applicability to that later work. |
| Packet | Internal wording for the prepared package of one review cycle: its batches, review units, attributed surrounding context, digests, lane assessments and pre-generated dispositions. The package is transient: it belongs to its cycle, is assembled from references rather than copies, and is frozen when published, so later information never rewrites it (5.5). Dispositions are recorded separately and never regenerated. It is not another name for a review unit, supporting source, disposition or cycle. |
| Later review information | Comments and dispositions append to the history of the cycle whose frozen package they reviewed, and do not rewrite it. Changed work or newly selected surrounding context after the review basis is frozen belongs to the next cycle. Comments, including those on approvals, become context for the next cycle (4.11). Inner-loop correction iterations happen within a cycle and do not count as cycles. |

## Retrieval and investigation

The [retrieval context and investigation foundation](foundations/11-retrieval-context-and-investigation.md) describes the retrieval layers, build order and context-reference contract, and marks proposed precomputation work. These definitions do not approve an allocation algorithm, a universal response ceiling, or new production capabilities.

| Term | Meaning |
| --- | --- |
| Chunk | A piece of a source file that the search index stores and returns whole. Keyword search and embeddings index chunks, and each carries identity and provenance (repository, revision, blob, path, lines and the chunker recipe) so a result can be cited. Boundaries follow the file's structure where it is known and line windows elsewhere; the recipe is recorded with each chunk. A chunk divides indexed source for retrieval; it is not a review unit, which groups changed lines by responsibility. Rank, size or being retrieved does not make a chunk evidence. |
| Ranked source evidence | Source regions selected and ordered for relevance to a query, with traceable locations. Sources can include code, tests, and prose. Rank does not establish correctness, approval, exhaustive coverage, or sufficiency for a review obligation. |
| Query context budget | A caller-selected byte allowance for the provider's budgeted query context. It bounds returned context, not graph storage or the extent of repository knowledge. The provider defines the counted representation and reports allocation loss. Bytes are not model tokens, and the budget does not necessarily cover the whole response envelope. |
| Source context expansion | Enlarging a returned source region or adding supporting context to explain a result. Expanded context can be necessary for understanding or verification; expansion does not make it optional by definition. |
| Contained callable identity | Provider-qualified identity and declaration-range metadata for a complete callable wholly visible within its owning result's primary source snippet. It identifies already-rendered source without duplicating it. Source presence and identity coverage are separate observations. |
| Required source evidence | Working meaning: the traceable source needed to satisfy a stated retrieval or evidence obligation. It describes necessity, not relevance rank. No general protected minimum or required-versus-discretionary allocation rule has been established. |
| Dependency depth or traversal depth | The number of relationship edges traversed from a selected starting entity under a stated direction, relation selection, and repository view. Direct neighbors are one hop away. Graphs may contain cycles and multiple paths. A requested or observed depth does not establish complete impact or sufficient evidence. |
| Blast radius | The potentially affected entities or review work identified through a scoped dependency and change-context analysis. The scope, relationship semantics, coverage, and limitations must remain visible. A structural connection or historical co-change is a contextual signal, not proof of a runtime effect. For a changed section it is a composite of measured parts, never one score: callers reached (for removed code, what still called it), dependency links (files that import the changed file, and links between changed symbols) and the size of the connected group of the dependency graph the change sits in. Each part is recorded with how its rows were matched and what it covers; a part that could not be measured is disclosed as missing and never read as zero. Tests count toward no part. It can raise a lane; it never lowers one and never lifts a gap. The connected group's size is not measured yet (14.22). |
| Impact finding | A finding that states what a change does to behaviour that already existed: removed, disabled, narrowed, changed or added. It says what changes, not how far the change reaches; how far it reaches is the blast radius. |

## Responsibility and review flow

The [review modes and sequencing](foundations/02-review-modes-and-cycles.md) define obligations and the human gate.

| Term                   | Meaning                                                                                                                                                                                                                                                                                                                                                               |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Review lane            | A responsibility category: static automation, agentic review, human-suggested review or human-required review. The harness assigns each unit a lane by assessing where it falls on the complexity × ease-of-validation matrix below (4.2). History, prior bugs, outages and blast radius can move a unit. A unit qualifies for static automation when checks meet its validation requirements (4.3). Assessment is best effort: when gaps prevent a confident assessment, the unit goes to human-suggested, with no special label (4.4). Weights and thresholds are not yet set. See [escalation and decomposition](foundations/06-escalation-and-decomposition.md). |
| Validation requirement | A statement of what must be observed to confirm that a review unit's changed behaviour is correct, drawn from its scope (for example a "Done when" item) or implied by the change. It is met by observed proof: for browser-facing work, logs (server, browser, network), screenshots and video, as the Chrome DevTools MCP captures them; for server-facing work, telemetry and logs. Every applicable kind must agree on the same result; one kind alone is not enough (11.1). Custom scripts are not proof. Tests are a base requirement, verifying the right things at the right boundaries, including error and input handling (11.2), kept to the minimum good practice needs; many tests in generated code is a warning sign, and bogus or forged tests or proof are defects. Coverage of each requirement is solid, shallow or missing, and the reader sees it as its [checkpoint](#responsibility-and-review-flow)'s proof state. A requirement is not itself evidence (10.58). |
| Checkpoint | A validation requirement as the reviewer sees it (10.110): one thing a review unit's change must satisfy, from its ticket or plan or implied by the change. The unit's briefing names it, and it appears as a contextual marker on the owned lines that meet it. Its anchors are fixed when the batch is created, and any external source it rests on is pinned then (11.17). Code sets its proof state from the proof records it rests on and their currency for the unit, in this order (11.16): Missing, no record serves it; Out of date, a record it rests on is no longer current (an owned file changed after the record's commit, that commit left the head's history, or an external source changed or could not be checked); Conflicting, the records it rests on disagree (11.19); Proven, two or more kinds of proof, every applicable kind recorded, a current test run among them wherever the behaviour can be tested (11.20), all current and in agreement; Thin, anything else. A checkpoint is unmet when it is Missing, Out of date or Conflicting; Thin is not unmet. A ticket item no requirement covers is shown as a Missing checkpoint (11.21). Every run checks again whether its proof is still current (11.18). It is not evidence, a verdict or a task for the reviewer. |
| Human-suggested        | Review that arrives with an agent disposition already completed before the human stage. A human may add judgment or request changes. Without human input, the existing agent disposition applies.                                                                                                                                                                     |
| Human-required         | Review that requires a human decision. The agent prepares a suggested review and comments, and may request changes, but never approves (4.5, 4.6). Neighbouring automated work does not discharge that obligation. |
| Disposition | An agent- or human-provided approval or request for changes for the review unit's owned work in the given cycle, including a comment that indicates further work is needed. Preserve the actor, exact reviewed source and cycle attribution. Supporting context does not receive or transfer that disposition. Rules for distinguishing informational comments remain open. |
| Inner development loop | The automated implementer and adversarial-reviewer process that improves the work before final review. The reviewer is responsible for change quality and evidence validity, judging converging proof against the plan's scope and definition of done (11.1, 11.3), requires corrections within the loop, and must approve before work advances to final review. Its quality and acceptance metrics remain separately observable within the broader cycle.   |
| Review cycle           | One pass from scope gathering, through the inner development loop and the harness pipeline, to review (4.11). It ends when the outer reviewers (human, agent or both) have given every needed disposition, approval or request for changes; comments alone do not end it. The next cycle then begins, and comments, including those on approvals, become its context. Inner-loop correction iterations do not count as cycles. Any disposition after the batch's first full set runs in a fresh context that reads the frozen package (4.7). |
| Human gate             | The final human review stage after automated processing for the submitted revision. Requests for changes return affected work to the development loop.                                                                                                                                                                                                                |

Starting quadrants for review lanes (4.2):

| | Easy to validate | Hard to validate |
| --- | --- | --- |
| **Low complexity** | Static automation or agentic | Human-suggested |
| **High complexity** | Agentic or human-suggested | Human-required |

## History and economics

The [history and metrics foundation](foundations/03-unit-history-and-metrics.md) defines attribution, while the [product purpose](foundations/01-product-purpose.md) establishes the cost-benefit commitments.

| Term               | Meaning                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Revision           | A change attributed to review work and its surrounding context. Corrections are connected to review units, each of which may have multiple requests or comments. Revisions accumulate into cycle metrics; exact deduplication rules remain open.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| Regression history | Evidence about later problems related to previously reviewed work, using Git and external sources. Confirming those relationships remains a design question.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| Reviewer's tax     | The effort human and agentic reviewers spend gathering, reconstructing, and repeatedly processing the information needed to assess work. Repetition means unnecessary revisiting of resolved units and their context. Each cycle builds on prior evidence; supporting-context overlap with new units is acceptable. Generated briefings provide material support and connected context for decisions. See the [reviewer's tax principle](foundations/05-reviewers-tax.md); its established dimensions and primary measures are unnecessary repetition, wrong attribution, and missing attribution. Attribution quality is a primary system-wide quality metric, used to improve evaluations and the graph. Exact measurement methods remain open. |
| Context tax        | The cost of supplying and processing context in the workflow, especially the inner loop. Track model input and output tokens separately; the full measurement definition remains open.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |

## Review interface

These terms name the surfaces, elements and actions of the Neuve web review UI. They were agreed through design rounds 1–3 (2026-10-03 to 2026-10-06; see the [design workflow](design/neuve-web/README.md) and the [round catalogues](design/neuve-web/changes/)). Visual rules live in the design sources. These entries fix meaning only, and none of them confers approval. Where a label is still a placeholder, the entry says so.

### Surfaces

| Term | Meaning |
| --- | --- |
| Batch overview | The screen for a current review batch. It takes the place of the pull-request page, as part of Neuve's [rethinking of the pull-request concept](foundations/04-review-units-and-context.md). It holds the isolated briefing, the lane bar, the units that need the reader's judgment, and verification. It starts review but records no disposition. |
| Review desk | The screen where one review unit is assessed and its disposition recorded. It holds the unit rail, the unit's contextual briefing, the diff, the context margin and the decision dock. |
| Unit rail | The desk's left column. It lists the batch's review units, grouped by review lane. It has two views: Units, and Files (the current unit's changed files). It shows the human-gate meter for units that still need the reader's decision. It can be collapsed. |
| Context margin | The column beside the diff that holds marker pins and their disclosure layers. Context stays out of the code until it is asked for. |
| Decision dock | The control at the foot of the desk that records the open unit's disposition. Its actions depend on the lane (13.25). Wherever a person can request changes, the request carries a comment, because that comment is the change being asked for (10.69). On human-required work, a person approves or requests changes, and a comment on an approval is optional. On human-suggested work, a person can approve or request changes at any time, and their decision overrides the agent's; approve shows as agree when the agent approved. On agentic work, a person can't approve, because approvals there are the agent's to give: people either delegate or disprove. They can request changes, or add a comment on its own to give the unit's next cycle context. On static-automation work, the dock shows only a red or green status, with no approve, comment or request changes for now. Every human-suggested, agentic or static-automation unit shows its lane's judgment (13.23). When the run has none, the dock says so: "no agent decision" on a human-suggested or agentic unit, "no check result" on a static-automation unit. |
| Review delivery | How much the review desk explains before the reader reads the code: one dial of four stops (names proposed: Facts, Briefed, Explained, Guided), set per workspace and refined by repository language. Higher stops add explanation around the same cited facts. At the last stop the diff gives way to the ticket's "Done when" items, and on human-required work a [suggested review](#surfaces) follows. It never changes the facts, the lane, the dock's actions or who decides. A reader can move it for one unit; that choice is not recorded. It never infers a reader's proficiency. Approved by the user on 2026-10-14 ([round 4](design/neuve-web/changes/round-4.md)). |
| Suggested review | A review the harness prepares for human-required work and shows at the Guided [review delivery](#surfaces) stop, reasoned from the ticket's "Done when" items and the evidence behind them. It is not a disposition until a person sends it, and sending records that person as the actor. On human-suggested work the agent's recorded review is shown instead. Internally a "suggested disposition". Decided by the user on 2026-10-14. |
| Lane bar | The batch overview's count of units per review lane. It shows counts, not dispositions. |
| Agents control | The header control that lists the agents connected to a branch, grouped as implementers and reviewers, each with scope and status. Connect picks from configured agents only. Disconnect drops the agent's context but keeps its thread history. |
| Lineage | The view of how a revision relates to earlier ones and what it asks of the reader. A recorded relation is an attributed claim, not transferred approval. |

### Context elements

| Term | Meaning |
| --- | --- |
| Marker pin | The diff-margin mark of the contextual markers attached to one line. There is one pin per line, except that markers in collapsed lines collect on the gap row. The pin shows the highest-priority marker kind and counts the rest. Review comments and drafts use the same pin anatomy in the margin, but they are not contextual markers. The [marker identity](design/neuve-web/context-markers.md) defines kinds, icons and families. A pin is a pointer to context, not evidence of correctness. |
| Marker kind | A contextual marker's category as the reader sees it: conflict, decision signal, failed check, assessment gap, unavailable evidence, same batch, contextual signal, test, captured source, graph, history, doc, checkpoint. Colour is reserved for kinds that ask something of the reader. Magenta means a decision is needed (conflict, decision signal). Amber means evidence is missing or failed. A dashed edge marks transient context or unsaved work. Exact ingestion rules per kind remain open. |
| Disclosure layers | The four depths of a marker. L0 hint: the pin. L1 peek: a short preview, one row per item on that line, on real hover or keyboard focus. L2 context card: the fragment, why it applies here, and what to check before deciding. L3 source sheet: the whole source, with the fragment highlighted and the other units that cite it. Each layer answers one more question, and the code never moves. |
| Citation chip | An inline reference in a briefing to the cited lines or marker. Activating it lands on that range. |
| Owned range | A changed range a review unit is responsible for, when the unit has ranged responsibility. Owned ranges are recorded with the unit in the [record log](architecture/decisions/0004-persistence-and-range-immutability.md). Owned ranges render in line order. The unit's disposition covers only its owned ranges. A unit that owns whole hunks lists each hunk's full range. |
| Gap row | A collapsed run of lines between owned ranges: unchanged lines, and any [sibling-owned lines](#review-units-and-context) folded in with them. It opens in place, part by part. It is context, not responsibility. |
| Sibling-owned lines | Changed lines shown in a unit's file that belong to another review unit's responsibility. They are read-only here, link to their own unit, and never receive this unit's disposition. They differ from a [read-only neighbor](#review-units-and-context): a read-only neighbor is a whole neighboring hunk from another lane, while sibling-owned lines can share a hunk with this unit's own ranges. |
| Sticky file header | The file's header, which stays at the top of the diff until its file scrolls out. It carries the file's position in the unit, layout controls and the viewed toggle. |
| Before you decide | The context card's single plain sentence naming what to check in real code or UI, with one action. It is advice, not an obligation record. |
| New-commit notice | A Git-sourced notice that the branch received a new commit while a review is open. Commits inside the inner development loop appear as thread activity, not as a new review cycle. |

### Comments and threads

| Term | Meaning |
| --- | --- |
| Review (in UI copy) | The name UI copy uses for a [disposition](#responsibility-and-review-flow), a verdict on a review unit. Code, documentation and records keep "disposition". This is a naming rule only: review comments keep their own name, and which comments count as requests for changes remains open (see Disposition). Compounds such as review unit, review batch and start a review keep their own meanings. Decided by the user on 2026-10-14. |
| Review comment | A comment anchored to lines of a review unit, or to the whole unit (10.102) and recorded in the [record log](architecture/decisions/0004-persistence-and-range-immutability.md). Records treat it as informational until it is explicitly recorded as a request for changes or referenced by a disposition. The general rules for which comments count as requests for changes remain open (see Disposition). Carrying it into later cycles as a contextual marker is a proposal. |
| Draft | An unsaved comment that is set aside, rather than discarded, when the composer closes. It is local to the reader and not recorded. |
| Thread | The conversation between the person reviewing and the agent working on a unit, started from a review comment. Its states are queued, sent, picked up, working, replied, needs your input, and failed. There is no read receipt. Only the agent's final reply counts as a message; interim narration collapses under working. |
| Implementer, reviewer | The two agent roles of the [inner development loop](#responsibility-and-review-flow), as shown in the agents control. A branch can have several of each, each with a path scope. |

### Actions

| Term | Meaning |
| --- | --- |
| Peek, open, drill | Move a marker from L0 to L1, L2 and L3 in turn. Peek happens on real pointer hover or keyboard focus only. |
| Land | Bring a target range into view below the sticky file header and tint it once, so the eye finds it. It changes nothing in the review state. |
| Mark viewed | Record, for the reader only, that a file has been looked at, and fold the file. It is progress, not a disposition, and it is not part of review history. |
| Agree | Accept an agent's existing approval on human-suggested work; requesting changes overrides it instead. Agreeing is recorded and shown as your approval ("You approved"), both at the moment of deciding and in history. Decided by the user on 2026-10-06 ([round-3 decisions](design/neuve-web/changes/round-3.md#shell-and-smaller-refinements)). |
| Start a review | Ask the assigned harness to begin review of the current batch. Starting a review doesn't approve the work. |
| Connect, disconnect | Attach or detach a configured agent for a branch. |

## Runtime

| Term | Meaning |
| --- | --- |
| Capacity | The machine-wide budget for Neuve's heavy work (10.93), counted in threads: one unit is one thread allowed to run. `neuve capacity` shows the threads each job holds and what waits. It bounds CPU, not returned context; the *query context budget* is unrelated. |
| Sandbox | A coding channel's own isolated place, where its shell and file tools run (12.28). It runs on the person's device for a channel with one person, and in the cloud for a channel with more than one. It holds the chat's worktree and what the sandbox image provides. Ace calls the cloud one a workspace; in Neuve it is always a sandbox, because *workspace* means Ace's collaborative home (12.7). |
| Vault | Where Neuve keeps the user's credentials and project settings (12.29): the macOS Keychain locally and Cloudflare's secrets store in the cloud. Settings and Records hold only names, never values. Neuve creates and owns no credentials of its own. |
| Broker | A sandbox's only way out (12.29). It swaps in the vault's real value for a stand-in only on requests to that secret's own hosts, answers Git only for the chat's own repository, and labels secrets in what leaves without blocking (12.30). Its lifetime follows its sandboxes. |

## Review feedback

| Term | Meaning |
| --- | --- |
| Feedback | The owner's answers in the review desk on a unit's lane and axes, on findings, on context references and on digests, recorded in Records (10.100). It is the owner's instrument for the climbs, shown behind a setting; it changes nothing in the review, and each answer records whether the harness's call was visible. |

## Vocabulary discipline

Use these terms consistently and link their definitions when needed. Add or revise a definition as the corresponding product decision becomes clear. Mark unresolved meanings explicitly; do not let a convenient implementation silently redefine a product concept.

---

# 附录：apps/neuve-cli/SETUP.md

> 来源：https://github.com/NeuveAI/neuve/blob/main/apps/neuve-cli/SETUP.md（README 引用的子文档，原文）

# Install Neuve in a repository

Most people install a release with the [README's one-line install](../../README.md#quick-start), which puts `neuve` in `~/.neuve`, and go straight to [setup](#set-up-a-repository). To work from a Neuve checkout instead, build the current Graph source, then install copies of the public CLI, that Graph engine and the review desk from your Neuve checkout into a folder of your own, and put its `bin` on your `PATH`:

```sh
bun run build:graph
bun run install:cli --prefix ~/.neuve-src
export PATH="$HOME/.neuve-src/bin:$PATH"
```

Keep a from-source copy out of `~/.neuve`, where the one-line installer puts release builds; it refuses a folder that holds a from-source install. Without `--prefix`, `install:cli` writes the machine's shared copy in `~/.local`, which every hook and terminal on the machine runs. That install is guarded: it takes only a clean integration branch such as `main`, after `bun run check` passed on that exact commit, so a half-tested build never replaces the copy other sessions use. Maintainers install that way; a copy in your own prefix skips those guards.

Graph's `mise.toml` selects its Go toolchain. `mise run build` is the equivalent configured build when mise is installed. The [root command table](../../docs/development.md#commands) says where `install:cli` puts the copies and what it refuses to replace; rebuilding or testing the workspace does not change them. If its manifest `<prefix>/lib/neuve/install.json` is corrupt or incomplete, remove it, `<prefix>/bin/neuve` and `<prefix>/lib/neuve/entire-graph`, then install again.

## Set up a repository

Run setup in an existing Git repository:

```sh
neuve setup --repo /absolute/path/to/repository
```

Repositories set up before the copy install keep their Graph link to the workspace build until `neuve setup --graph-binary <prefix>/lib/neuve/entire-graph` is re-run in them, with the prefix you installed into (`~/.neuve-src` above, `~/.local` for the shared copy).

Run setup before a review. In a repository where setup hasn't run, `neuve review` and `neuve review desk` stop and print the `neuve setup` command to run. Setup resolves the Git root, records the built executables it found and checks, through the validated Graph-client boundary, that the Graph engine can work on the repository: its identity and capabilities, the repository's Git metadata and the engine's preparation plan for `HEAD`. In a repository whose objects live in another repository (cloned with `--shared` or `--reference`), the client doesn't read the Git metadata yet, so setup checks the engine's identity and capabilities alone; reviews read their own clone and work there. It indexes nothing: a review builds and stores what it reads itself. It reports whether local work differs from `HEAD`.

Setup keeps its state beside the Records store, in `<git common directory>/neuve/setup/`, so the repository's working tree and `git status` are the same before and after it. Every worktree of the repository shares it, as they share the store.

| Path under `<git common directory>/neuve/setup/` | Purpose                                                                                                    |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------- |
| `config.json`                                    | The installation (schema 4): the repository that ran setup, and the CLI and Graph engine it found, by path |
| `initial-index.json`                             | The engine check: its status, the revision and the engine's identity                                       |

Setup links nothing: the Graph engine refuses a repository whose Git directory holds a link to another volume. A recorded engine that is gone (after an install elsewhere) gives way to the engine beside the CLI the next time setup runs. A Graph refresh (`neuve query --refresh-cache`) checks the revision out in a temporary view outside the Git folder and on the repository's disk: `<temporary folder>/neuve-<uid>/graph-views/`, else `/private/tmp/neuve-<uid>/graph-views/`, else `.neuve-<uid>/graph-views/` at the top of the repository's disk, each a private folder of yours, removed when empty. Only when none can be made does it go in the store's `graph/views/`, under the same checks, where Graph refuses a repository that commits a link leaving its disk; `neuve query` then says so and what to change.

Setup also settles how reviews sign in, kept as `review.route` in your Neuve settings (`~/Library/Application Support/Neuve/settings.json` on macOS):

- `neuve setup --route openai` chooses ChatGPT; then sign in with `neuve auth login openai`.
- `neuve setup --route anthropic` chooses an Anthropic API key; then set `ANTHROPIC_API_KEY` in the shell you run Neuve from. It must be an API key: a Claude subscription token doesn't work.
- `neuve setup --route claude` chooses your Claude subscription, through Claude Code. Install Claude Code and sign in with its own `claude auth login` first; setup then checks that sign-in, as `neuve auth status claude --check` does, without spending usage. Both need Claude Code installed.
- Without `--route`, setup asks on a terminal when no route is set yet ("How should reviews sign in?", with the three choices). With no terminal (a script or an agent) it asks nothing, writes nothing and says how to choose. A route already set is changed only with `--route`.

Setup changes only `review.route`: every other setting stays as it was. It refuses a settings file it can't read rather than overwrite it. When the settings path is a link (settings kept in a dotfiles folder), it writes the file the link leads to and keeps the link; a link that doesn't lead to a file of yours is refused. It never signs in for you. `neuve doctor` shows the route and whether its sign-in is there, without reading the credential. For the Claude route it only checks that Claude Code is installed, because checking the sign-in opens Claude Code, which can use the network.

`neuve review` and `neuve review desk` refuse a repository until setup has run there. Records live in `<git common directory>/neuve/` too; `neuve records --help` describes them. Graph's engine copy and the snapshots `neuve query` builds live in the machine's shared Graph cache, `<budget folder>/graph/cache/`, the one reviews use. `--graph-binary` and `--cli-binary` explicitly select built executables.

An earlier setup wrote `config.json`, `initial-index.json` and the links `bin/neuve` and `bin/neuve-graph` to `neuve/` in the working tree and added `/neuve/` to the local Git exclude file. Neuve's own record there still counts, so such a repository stays set up. The next `neuve setup` removes that folder only when every part is provably Neuve's: real folders, its own record naming those links and this repository, links where it names them, an index it wrote, and nothing Git tracks. Otherwise it leaves the folder as it is and says so in one line; a project's own `neuve/` folder never stops setup. The exclude line stays; it is harmless. Files from the removed review stack (`neuve/backend.env`, `neuve/acp-initialize.json`, `neuve/ledger.sqlite`, `neuve/cache/graph/`) are not read, and you may delete them.

## Review settings

Your own [review settings](../../packages/harness/src/settings.ts) choose the model a review runs on. They live outside every repository, at `~/Library/Application Support/Neuve/settings.json` on macOS and `~/.config/neuve/settings.json` elsewhere. Without the file, reviews run on the Anthropic route.

```json
{
  "version": 1,
  "review": { "route": "openai", "effort": "medium" }
}
```

| `review.route` | Model | What it needs |
| --- | --- | --- |
| `anthropic` (the default) | Claude Opus 5.5 through the Anthropic API | `ANTHROPIC_API_KEY` in the environment of the worker that runs the review |
| `openai` | GPT-6.1 Sol on your ChatGPT plan | `neuve auth login openai` once; `neuve auth status` shows the sign-in |
| `claude` | Claude Opus 5.5 on your Claude subscription, through Claude Code | Claude Code installed and `claude auth login` once; `neuve auth status claude --check` checks the sign-in |

`claude` runs reviews through your installed Claude Code and its own sign-in. Neuve never reads that sign-in. `neuve review <base>..<head> --route claude` does the same for one review. `neuve auth add claude <name>` adds a second Claude sign-in in its own folder and prints the command to sign in; review on it with `--route <name>`. An added account can't take the name of a route or a built-in account (`anthropic`, `openai`, `claude`, `chatgpt`).

If you installed Neuve before `claude` was a route, update it first (`neuve upgrade`, or the one-line install again if your `neuve` has no `upgrade`, or `install:cli` again for a checkout copy): an older `neuve` refuses a settings file that names `claude`.

`review.effort` is `low`, `medium` (the default) or `high`. A review queued where its route's credential is missing waits, and `neuve review` says so; start a worker from a shell that has the credential with `neuve worker start --repo <path>`.

## Open the review desk

```sh
neuve review desk [--repo <path>] [--port <n>]
```

This opens the batch overview and the review desk for the repository you're in, or for the one `--repo` names. It prints the desk's address, such as `http://127.0.0.1:4141/`. The desk stays open until you press Ctrl-C. If port 4141 is taken, it picks a free port and says so. `--port 0` always picks a free port. The desk listens on this computer only.

Each repository has one desk. If its desk is already open, `neuve review desk` prints that desk's address. While the desk is open, `neuve review` prints a link to the review. Otherwise it says to run `neuve review desk`.

The desk is installed with the CLI, in `<prefix>/lib/neuve/desk/`, and runs inside `neuve`, so it needs no Bun. If that folder is missing, incomplete or from another version, the command stops and names the command that installs it again. To run `neuve` from a Neuve checkout without installing it, set `NEUVE_DESK_DIR` to a built desk folder, such as the `lib/neuve/desk` folder of an install.

## Check what's missing

`neuve doctor [--repo <path>] [--json]` checks, without changing anything or using the network, what a first review needs: the CLI and the files its install recorded, `PATH`, the Graph engine, the Xcode command-line tools on a Mac, Git, the repository's setup, the review route's credential, the repository's worker, the review desk, and whether the installed Neuve skill is older than this `neuve`. Each line is `ok`, `missing` or `warning`; a missing or warning line ends with the command that fixes it. It names credentials, never their values. It exits 1 when something required is missing. `neuve setup` ends with the same credential check and the next command to run.

## Capture commits and agent sessions

`neuve setup hooks --repo <path>` enables capture for one repository. It installs `post-commit` and `post-rewrite` hooks in the repository's hooks directory, chaining any hook already there (kept as `<name>.pre-neuve`). It writes `<git common directory>/neuve/capture.json`. It also adds Claude Code and Codex hooks to your user configuration (`~/.claude/settings.json`, `~/.codex/hooks.json`), backing up any file it changes as `<file>.neuve-backup-<time>`. Agent hooks are per user, but they do nothing in repositories without `capture.json`. Run with `--dry-run` to see the changes first, and rerun it to update them.

Hooks run a small hook entry: the installed `neuve hook`, or, when setup runs from a checkout, a bundled snapshot of that entry in `~/Library/Application Support/Neuve/hook/neuve-hook.js`, so source being edited cannot break them. Rerun setup to refresh it. Each hook command discards its output and always exits 0, so it never adds to an agent's context or blocks a prompt or a commit. Failures are logged to `<git common directory>/neuve/hooks.log`. Hooks only write events to `<git common directory>/neuve/spool/` and wake the worker, which ingests them into Records. Prompts never enter recorded events. At each turn's end and at session end, hooks record the transcript's path and size; the worker then copies the transcript segment since the previous turn end into the store as a raw capture (it holds the prompts and tool output as the agent wrote them) and records the turn's model, token counts, model calls and tool calls. Raw captures are pruned after 30 days and the token figures are kept ([retention ruling 8.10](../../process/decisions.md#area-8-records-rebuild-m2-17-october-2026)); `neuve records stats` shows them. A commit is linked to the agent session that made it through process ancestry. When ancestry is unavailable, as in a sandbox, the harness's own session variable (`CLAUDE_CODE_SESSION_ID`, `CODEX_THREAD_ID`) links it as `declared` when it names a live session recorded in this repository ([declared-link ruling 8.13](../../process/decisions.md#area-8-records-rebuild-m2-17-october-2026)). `NEUVE_SESSION=<harness>:<session id>` names the session explicitly. An unlinked commit is attributed to its committer. `neuve agents` lists actors with their aliases, models, turns and commits. `neuve agents current` shows which session a call belongs to.

Pruning leaves a tombstone, so records that reference a capture stay intact. The worker prunes at most once a day, so a capture goes within a day after its 30 days. `neuve records prune --dry-run` shows what a prune now would remove, by class and age; `neuve records prune` enqueues one and wakes the worker.

`--session-trailer` adds a `Neuve-Session` commit trailer to linked commits in your repository. It is off by default and refused in the Neuve repository ([commit-link ruling 8.1](../../process/decisions.md#area-8-records-rebuild-m2-17-october-2026)).

## Let agents vent

An agent may say, once per session, what is making that session difficult ([venting ruling 8.8](../../process/decisions.md#area-8-records-rebuild-m2-17-october-2026)). A vent never changes the task or a review. `neuve vent "<text>"` records one from a shell command; `neuve vents [--since 7d] [--agent <alias>] [--run <id>] [--json]` lists them, with control characters and terminal escape sequences removed from what agents wrote. A vent is tied to its agent session the way commits are: through process ancestry, the session ID the harness declared, or `--session <harness>:<session id>` (or `NEUVE_SESSION`). With no session to tie it to, nothing is recorded and the command says why. A second vent in the same session is answered normally, and the first is kept. A harness that runs several agents under one session passes each review cycle's session ID as `--reconciliation-id <id>` (the MCP tool's `reconciliationId`), as the [vent-attribution ruling 8.16](../../process/decisions.md#area-8-records-rebuild-m2-17-october-2026) sets out. Vent text is pruned after 30 days ([vent-retention ruling 8.17](../../process/decisions.md#area-8-records-rebuild-m2-17-october-2026)), and `neuve vents` then shows it as pruned. Vents recorded with the earlier inline-text shape (`vent.recorded@1`, from the S6 build only) keep their text, because the log is append-only.

`neuve mcp` serves the same `vent` tool, and `record_proof` for proof records, over MCP on stdio. The harness starts it as a subprocess for each session and it exits when the session closes its input: no port and no daemon. Setup does not install it; add it yourself, once per user:

```sh
# Claude Code (user scope, every repository)
claude mcp add --scope user neuve -- neuve mcp
```

```toml
# Codex: ~/.codex/config.toml
[mcp_servers.neuve]
command = "neuve"
args = ["mcp"]
```

The server works in the repository of its working directory, normally the one the harness was started in; pass `--repo <path>` to fix one. Each call is tied to the session whose agent process started the server, so the session's hooks (`neuve setup hooks`) must be installed for the repository. A server started outside a captured session answers each call that nothing was recorded.

Queries use the Graph engine setup recorded while it runs, else the one beside the CLI. An explicit `NEUVE_GRAPH_BINARY` overrides both; provider cache environment variables don't change where the cache is.

## Other commands

`neuve --help` lists every command, and `neuve <command> --help` gives its options.

| Command | What it does |
| --- | --- |
| `neuve upgrade [--version vX.Y.Z]` | Installs the latest release, or the given tag, through its checksum-checked install script |
| `neuve skills list\|install\|uninstall` | Installs the Neuve skill, which teaches Claude Code and Codex to use Neuve, for your user or in the repository (`--scope user\|repo`) |
| `neuve setup stack` and `neuve stack list` | Fetch, in the background, a read-only copy of the dependency sources the repository's lockfiles pin, and list what a revision pins |
| `neuve proof record\|list <base>..<head>` | Records observed proof for a range (test runs, type checks, logs, telemetry, screenshots, video) and lists it |
| `neuve worker start\|status\|stop` | Starts, inspects or stops the repository's background worker |
| `neuve jobs [list\|inspect\|drop]` | Lists and inspects the worker's jobs, or drops a waiting one |
| `neuve relations prepare <base>..<head>` | Stores the Graph relations of a range's two sides before its review |
| `neuve review show [<base>..<head>]` | Prints a run's receipt: its batches, units and findings |

## Dogfood the Neuve monorepo

Run dependency installation at the monorepo root; the root `bun.lock` governs the combined workspace.

```sh
bun install
bun run install:cli
neuve setup --repo /absolute/path/to/neuve
```

Running setup from a nested source directory resolves the same monorepo, so its record lives in the root's `<git common directory>/neuve/setup/`.

Start the web app from the root with `bun run dev`. The [web README](../neuve-web/README.md) describes it. It runs from source, never the installed copies.
