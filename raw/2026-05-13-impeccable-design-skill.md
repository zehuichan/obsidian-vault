---
url: "https://impeccable.style/"
clipped: 2026-05-13
title: "Impeccable: The missing upgrade to Anthropic's frontend-design skill"
---

# Impeccable

> Design fluency for AI harnesses

Great design prompts require design vocabulary. Most people don't have it. Impeccable teaches your AI deep design knowledge and gives you 23 commands to steer the result.

**What's included:** _Impeccable_ agent skill with 23 design commands · Optional CLI + Chrome extension

**Works with:** Cursor · Claude Code · Gemini CLI · Codex CLI · VS Code Copilot · Antigravity · Kiro · OpenCode · Pi

Current release: **v3.0.7** — italic-serif hero and eyebrow-chip detection, recoverable live sessions.

## Before / After

**Before — generic AI output**

> Welcome to Our Platform
> The best solution for your needs
> Lorem ipsum dolor sit amet, consectetur adipiscing elit. This is generic placeholder text.
> [Get Started]
>
> Inter font · Purple gradient · Generic copy · Cards on cards

**After — with design skills**

> Introducing
> Thoughtful Design
> Every element serves a purpose. Hierarchy guides the eye. Whitespace breathes.
> [Explore]

## 01 — The Foundation

Before any command runs, Impeccable teaches your AI how design works. **Seven reference files** load on every prompt:

- typography
- color
- motion
- spatial
- interaction
- responsive
- UX writing

Run `/impeccable teach` once to set your project's design context. Every command benefits.

## 02 — The Language

23 commands give you a shared design vocabulary with your AI. Each maps to one discipline:

- `/typeset` for type
- `/colorize` for color
- `/animate` for motion
- …and 20 others (full reference on the docs site)

The autocomplete shows the full list the moment you type `/impeccable`.

## 03 — The Case

Nine talking points the project organizes around:

1. Visualize, then ship
2. Intentional design
3. Prevents AI slop
4. Production codebases
5. Brand + Product design
6. Ships code, not mockups
7. Fluent in DESIGN.md
8. CI/CD-ready
9. Chrome extension

### Design that knows who it's for

Most AI tools one-shot a plausible-looking mock. Impeccable starts with a conversation: audience, brand personality, anti-references, aesthetic direction. It captures the answer in `PRODUCT.md` and `DESIGN.md` and loads them on every command. The output is design that fits the actual business, not a generic impression of one.

**PRODUCT.md (loaded on every command)** — example:

- **Users**: SREs on call, reading fast, often in the dark.
- **Register**: Product. Design serves the task.
- **Brand voice**: Calm, clinical, no hype.
- **Anti-references**: Purple gradients. Glassmorphism. "Boost your productivity."

Commands involved: `/impeccable teach`, `/impeccable document`, `/impeccable shape`.

### Picture it. Build it. Ship the page.

Case study "Neo Mirai" started as generated reference images: a brand toolkit and a hi-fi conference mock. `/impeccable craft` turned them into a real responsive website, then iterated in the browser until the live page matched the direction.

1. **Visualize** — Auto-generated hi-fi landing-page mock in editorial warm-paper style.
2. **Shape** — Auto-generated brand toolkit: identity lockups, palette, type, icon system, and application mocks for a fictional AI design conference.
3. **Live build** — Finished Neo Mirai website hero and agenda, implemented from the generated conference mock.

A generated reference became shipped HTML, CSS, assets, responsive fixes, and interaction polish.

### Every model learned from the same slop

Without intervention, every AI model produces the same predictable mistakes: purple gradients, nested cards, gradient text, low-contrast labels. Impeccable names 25 of them, detects them in your code and in the browser, and teaches the AI to avoid them.

**Gallery of Shame:**

- Purple Gradients
- Side-Tab Cards (thick border accent)
- Cardocalypse (cards nested inside cards)
- Inter Everywhere
- Template Layouts
- Bad Contrast

### Built to live in your production codebase

Impeccable reads your existing tokens, components, and rendered output. It polishes what's there, flags regressions, and stays inside your design system instead of inventing a new one each run.

Example `/impeccable polish` run:

```
› Walking src/…
✓ src/styles/tokens.css     14 colors, 6 font sizes, 7 spacing steps
✓ tailwind.config.ts        theme.extend merged
✓ src/components/           34 components
   · Button: primary, secondary, ghost, destructive
   · Card:   3 variants, 8px / 16px radii
   · Input, Select, Dialog, Toast, Table…
✓ DESIGN.md                 brand rules loaded
→ Matching your system. Not inventing one.
```

### Brand work is not product UI

Most design skills, including Anthropic's own `frontend-design` skill, collapse everything into one vocabulary. But a landing page and a dashboard live by contradictory rules. Impeccable runs in two modes:

- **brand** — design IS the product: marketing, portfolios, editorial
- **product** — design SERVES the product: app UI, tools, dashboards

Commands like `typeset`, `animate`, `colorize`, `bolder`, `quieter`, `layout`, and `delight` adjust their choices to match. The mode is inferred from your task and confirmed in `PRODUCT.md`.

### Design where the work actually happens

The waterfall era (designer in canvas, throws a mock over the wall) is over. Most product work now starts directly in Claude Code, Cursor, Codex, Gemini, or Copilot, on the real codebase, in the browser. Impeccable runs there. No canvas. No handoff.

### Your design system in a portable file

`/impeccable document` scans your codebase and writes a `DESIGN.md` that follows the **Google Stitch DESIGN.md format** (the emerging standard). Six fixed sections in a fixed order, parseable by every other DESIGN.md-aware tool.

Example DESIGN.md fragment:

- **Accent**: `oklch(60% .25 350)`
- **Display**: Italic 400, Cormorant Garamond
- **Body**: Regular 400, Instrument Sans
- **Components**: 34 (Primary, Ghost, …)

### Block AI slop before it ships

Run `npx impeccable detect src/` on a PR branch and get JSON output with every purple gradient, nested card, low-contrast label, and gradient-text heading caught. Fail the build on severity. **27 deterministic checks**, no LLM required, no API key, no flaky scoring.

```
$ npx impeccable detect src/
✗ src/Hero.tsx:42      gradient-text
✗ src/Card.tsx:8       side-stripe-border
✗ src/Button.tsx:17    ai-color-palette
3 anti-patterns · exit 1
```

27 rules · `--json` · `--fast` · exit codes for CI.

### One click on any site

The Impeccable Chrome extension runs the same 25-rule detector on every page you visit, in-browser, with zero setup. Audit your own staging, audit the design your PM showed in Slack, audit your competitor on a Tuesday. The overlay highlights each issue in place.

## 04 — Live Mode (Alpha)

Pick any element in the browser. Drop a comment or a stroke. Hit Go. Three production-quality variants swap in via your framework's HMR. Accept the one you want and it writes back to source.

- Start `/impeccable live` to open, then stop live mode when you're done.
- **Your agent, your code**: variants are generated as actual edits, not mocks.
- **Accept = write**: an accepted variant lands in your source file. Canvas-like iteration, production output.
- **Skill-native**: no external tools or IDE extensions. Works across Claude Code, Cursor, Codex, Gemini, and every harness the skill targets.

## 05 — Get Started

### 1. Install the skill

```
$ npx skills add pbakaus/impeccable
```

Works with Cursor, Claude Code, Gemini CLI, Codex CLI, and more.

**Other install methods:**

- **Claude Code plugin**: `/plugin marketplace add pbakaus/impeccable`, then open `/plugin`.
- **Manual download**: universal ZIP for all 12 providers.

### 2. Use it

```
/impeccable redo this hero
```

Let the skill pick the approach. No names to memorize.

```
/impeccable audit checkout
```

Reach for a command by name. Type `/impeccable` alone to see all 23.

```
/impeccable pin audit
```

Pin your favorites. `/audit` becomes a standalone shortcut.

### 3. Add the Chrome extension (optional)

Click the toolbar icon on any page and every anti-pattern lights up right where it lives. Catches gradient text, AI color palettes, nested cards, low contrast, and 20+ more rules. Works on localhost, staging, production, or anyone else's site.

### 4. Install the CLI (optional)

```
$ npm i -g impeccable
```

Then `impeccable detect src/` to scan a project, or `npx impeccable detect` without installing.

### 5. Stay updated

```
$ npx impeccable skills update
```

Run periodically to pull the latest skill definitions.

## 06 — What's New

Highlights since v3.0 (April 10 – May 4, 2026):

- **Live mode preserves identity by default.** Variants stay on-brand for the existing surface and explore different expression axes within that identity. Departure from the current aesthetic only triggers when PRODUCT.md anti-references explicitly reject the current surface, or the user asks for it. Departure directions derive from brand voice, not a fixed catalog.
- **Detector flags the new monoculture.** The overused-font rule now catches Fraunces, Geist, Mona Sans, Plus Jakarta Sans, Space Grotesk, Recoleta, and Instrument Sans. Contrast checks run on styled `<a>` and `<button>` elements. Brand exceptions for Vercel, Next.js, and GitHub on their own domains.
- **Live mode works everywhere.** Strict-CSP apps, modal hosts (Radix, Headless UI, vaul), React/TSX projects, and repeated identical-class siblings all handle cleanly through the wrap, preview, accept, and carbonize loop.
- **PRODUCT.md + DESIGN.md as shared context.** A strategic file (who, what, why) and a visual file (colors, typography, components in the Google Stitch format) that every command reads before generating. Created via `/impeccable teach` and `/impeccable document`.
- **18 skills became 1 skill with 23 commands.** One `/impeccable` entry in the menu, shared design vocabulary, and `/impeccable pin` to promote favorites back as standalone shortcuts.

### v3.0.7 — May 4, 2026

- **Detector flags italic-serif display heroes.** Oversized italic serif (Fraunces, Recoleta, Newsreader, Playfair, Cormorant, Tiempos) running as the primary hero h1 is now caught as a structural fingerprint of late-2025 and early-2026 AI-generated marketing pages. Editorial and magazine surfaces that legitimately want the pattern can ignore the rule.
- **Detector flags hero eyebrow chips.** The uppercase letter-spaced label sitting directly above a hero h1 now fires, including the pill-chip variant (background plus 999px border-radius). Bounded to short labels at small sizes so editorial captions and ordinary subheads do not false-positive.
- **Live mode survives disconnects.** A durable session journal records every event, so an agent crash, a network blip, or a browser refresh no longer loses the session. Three new sub-commands: `live status`, `live resume`, `live complete`.
- **Reference files stripped of repetitive scaffolding.** SKILL.md and 33 sub-command files lost the "Remember:" closer chants, brochure-style "[Verb] [object] to [outcome]" openers on 12 older commands, and 419 em-dashes. Less context per command load, less repetition the model has to read past.

### v3.0.6 — April 30, 2026

- **Live mode preserves identity by default.** Variants stay on-brand for the existing surface (same palette, type pairing, visual rhetoric) and explore different expression axes within that identity.
- **Departure mode derives from brand voice, not a fixed catalog.** The old lane list (Swiss-grid, Terminal, Industrial-signage, etc.) caused the model to converge on the same three directions every time. Replaced with a brand-voice derivation process.
- **Parameters ship consistently on large surfaces.** Planning params is now part of variant planning, not a separate step. 2-3 knobs per variant for heroes and sections.
- **Reflex-reject aesthetic lanes.** Editorial-typographic is the first entry, catching the second-order training reflex where every departure from SaaS-cream defaults to magazine-cover aesthetics.
- **Two-altitude category-reflex check.** Catches both first-order (theme/palette from category alone) and second-order (aesthetic family from category + anti-references) reflexes.

### Extension v1.0.3 — April 29, 2026

- **Contrast checks now run on styled buttons.** A styled `<a>` or `<button>` with its own opaque background was silently skipped; a "Get started" pill with charcoal text on a near-black background read as fine to the detector. Inline links inside paragraphs continue to skip.

### Extension v1.0.2 — April 29, 2026

- **Popup scan no longer hangs after a page reload.**
- **Detector flags the new monoculture.** Fraunces, Geist, Mona Sans, Plus Jakarta Sans, Space Grotesk, Recoleta, Instrument Sans. Brand exceptions for Vercel, Next.js, GitHub.

### CLI v2.1.8 — April 28, 2026

- **Detector runs on Windows.** CLI path resolution used `new URL(...).pathname`, which prepends a slash to drive letters (`/C:/...`) and breaks `fs` on Windows. Switched to `fileURLToPath`.
- **Border-radius detection no longer flickers under jsdom.**

### v3.0.5 — April 28, 2026

- **Live mode lands valid TSX through the wrap → preview → accept → carbonize loop on Vite/Next React/TSX projects.** The wrapper keeps a single JSX-slot child instead of three adjacent siblings, round-tripping cleanly inside `return (...)`, array `.map(...)`, and `asChild` parents like Radix's `<DialogPrimitive.Title>`.
- **Wrap correctly disambiguates repeated identical-class siblings.** `live-wrap.mjs` now accepts `--text TEXT` (the picked element's `textContent`) and narrows candidates accordingly.
- **`live-inject` CSP-meta unwrap now byte-for-byte preserves self-closing tag whitespace.**
- **`live.md` spec gained explicit guidance for three real authoring traps.** Descendant combinator (`:scope > .card`), JSX `<style>` template-literal wrapping, abort via `live-poll --reply EVENT_ID error` not `live-accept --discard`.

### v3.0.4 — April 28, 2026

- **`/impeccable craft` now treats approved mocks as visual contracts.** Requires a mock fidelity inventory before build; missing hero objects, imagery, section structure, nav/CTA treatment, and distinctive motifs become blocking defects unless the user accepted the deviation.
- **Image-led brand surfaces can no longer degrade into abstract panels.** Travel, editorial, portfolio, venue, product showcase, entertainment, and education work now requires credible imagery.
- **`/impeccable craft` and `/impeccable shape` hardened against autonomous agents.** Codex-class harnesses had started writing files before user confirmed a design brief, treating their own summary as the brief. Setup now ships a preflight checklist; craft enforces an explicit build gate (`shape=pass` only counts when the user separately approves the brief).
- **Live picker plays nice with modal hosts.** Inside Radix Dialog, Headless UI, vaul, and other portals that lock `body { pointer-events: none }` or attach outside-click dismissers, the picker chrome had become unclickable; now defangs outside-handlers at the chrome boundary.

### v3.0.2 — April 27, 2026

- **Claude Code plugin install shrunk by 380×.** The marketplace ships only the runtime payload (~770 KB) instead of the entire monorepo (291 MB). Plugin source moved from `./` to `./plugin`.
- **Slash commands now register reliably on Claude Code.** The `skills` field in `plugin.json` was missing the trailing slash that the documented schema expects.

### v3.0.1 — April 24, 2026

- **Live mode runs in strict-CSP apps.** When your HTML carries a CSP meta tag, `/impeccable live` silently appends the live-server origin to `script-src` and `connect-src` (and `blob:` to `img-src` for the screenshot overlay) on session start, and reverts the patch verbatim on stop.
- **Live mode survives conditional-render content.** Picking an element inside a closed modal, an inactive tab, or a collapsible panel no longer wedges live mode when Vite Fast Refresh remounts the parent.
- **Live mode no longer breaks JSX projects.** Three round-trip bugs in `/impeccable live`'s accept path are fixed.
- **`/impeccable polish` now a true superset of the retired `/normalize`.** Aligning to the design system is non-optional; drift is named by root cause (missing token, one-off implementation, or conceptual misalignment); a new Information Architecture & Flow dimension covers progressive disclosure, established user-flow shapes, and naming consistency.

### v3.0 — April 10, 2026

- **Live Mode (Alpha).** Run `/impeccable live` and iterate on your UI in the browser: pick any element, drop a comment or stroke, hit Go, get three production-quality variants swapped in via your framework's HMR. Works on Vite, Next.js (including monorepos), SvelteKit, Astro, Nuxt. Detects your project's CSP and offers a one-time, dev-only patch.
- **Visualize, then build.** Image gen crossed the reference-quality threshold with GPT Image 2, Nano Banana Pro, and Imagen 4 Ultra. `/impeccable shape` drafts a brand toolkit (color, typography, mood board) as real images; `/impeccable craft` pre-renders the hi-fi mock to code toward. Strongest in Codex with GPT 5.5.
- **PRODUCT.md, shared design memory for your AI.** A single file at your project root that names the audience, brand personality, anti-references, and register (brand vs product). Every command reads it before generating. Created in one step via `/impeccable teach`.
- **DESIGN.md generation, spec-compliant and interoperable.** `/impeccable document` scans your tokens, components, and rendered output and writes a DESIGN.md that follows the Google Stitch DESIGN.md format. Six sections in the exact order other DESIGN.md-aware tools expect.
- **Brand and product registers.** Design work splits cleanly into two worlds: brand (marketing sites, landing pages, portfolios) and product (app UI, dashboards, tools). Register-aware commands (`typeset`, `animate`, `bolder`, `quieter`, `colorize`, `layout`, `delight`) each adjust their vocabulary to match.
- **18 skills became 1 skill with 23 commands.** Every command now lives under `/impeccable`. One entry in your `/` menu instead of 18, a shared design vocabulary between you and your AI, and far less namespace pollution as the plugin ecosystem grows.
- **Pin your favorites back as shortcuts.** Run `/impeccable pin audit` and `/audit` becomes a standalone command again. Under the hood it writes a lightweight redirect skill that delegates to `/impeccable audit`.

### v2.1 — April 9, 2026

- **Streamlined from 21 to 18 commands.** Removed overlap: `/arrange` renamed to `/layout`, `/normalize` merged into `/polish`, `/onboard` merged into `/harden`, and `/extract` became `/impeccable extract`.
- **Automatic cleanup of deprecated skills.** On first load after updating, the skill detects and removes leftover files from renamed or merged commands.

### v2.0 — April 8, 2026

- **Renamed `frontend-design` to `impeccable`.** The core skill now shares its name with the project, and the teach subcommand moved from `/teach-impeccable` to `/impeccable teach`.
- **Skill rewritten against evals.** Fifteen briefs ran through gpt-5.4 and Qwen 3.6 Plus, with and without the skill loaded, then graded side by side. More font and color variety, fewer purple gradients, much better Codex output. **The single change that moved the numbers most: before the model commits to a font or palette, it has to name its first three instincts and reject them.**
- **Anti-pattern detection engine.** 27 deterministic rules across typography, color, layout, motion, and quality. Handles oklch, oklab, lch, and lab color formats, CSS variables inside border shorthands, gradient-backed text, and emoji-only nodes.
- **CLI: `npx impeccable detect`.** Scans HTML, CSS, JSX/TSX, Vue, Svelte, and CSS-in-JS. Framework detection, multi-file import tracking, Puppeteer-backed live URL scanning, CI-ready JSON output, `--fast` regex mode.
- **Chrome DevTools extension.** One-click detection on any page. Reads live computed styles, surfaces findings in an interactive panel, highlights elements on the page.
- **`/critique` got teeth.** Persona sub-agents review in parallel, score against Nielsen's heuristics, run the detector automatically, and open a live browser overlay.
- **New ways to create with Impeccable.** `/shape` runs a structured discovery interview about purpose, audience, and goals, then produces a design brief before any code is written. `/impeccable craft` chains that brief straight into the full implementation flow.
- **New harness: Rovo Dev.** 11 supported AI tools total.

### v1.6.0 — March 18, 2026

- New provider: **Trae** (China + International)
- `/critique` now scores against Nielsen's 10 heuristics, tests with persona archetypes, and assesses cognitive load
- `/audit` now scores 5 dimensions with P0-P3 severity ratings and structured action plans
- Improved skill descriptions for better agent auto-discovery
- Fixed invalid YAML frontmatter that broke GitHub preview and Codex loading (#67)
- Codex CLI now uses correct `$` prefix for command references

### v1.5.1 — March 17, 2026

- `/typeset` now recommends fixed type scales for app UIs, reserving fluid typography for marketing/content pages

### v1.5.0 — March 16, 2026

- 3 new skills: `/typeset` (fix typography), `/arrange` (fix layout & spacing), `/overdrive` (technically extraordinary effects, beta)
- Skills now auto-gather design context via `.impeccable.md`. Run `/teach-impeccable` once, all skills benefit
- Deep linking to commands (`#cmd-overdrive`, etc.)

### v1.3.0 — March 12, 2026

- Added OpenCode provider support
- Added Pi provider support
- Recategorized `/onboard` as an enhancement command

### v1.2.0 — March 5, 2026

- Added Kiro support (`.kiro/skills/`)
- Restored prefix toggle: download `i-` prefixed bundles to avoid naming conflicts
- Audit and critique skills only suggest real, installed commands

### v1.1.0 — March 4, 2026

- Unified skills architecture: commands are now skills with `user-invocable: true`
- Added VS Code Copilot and Google Antigravity support
- New install flow: `npx skills add` as primary, universal ZIP as fallback
- Added universal ZIP containing all 5 provider directories
- Renamed `/simplify` to `/distill` to avoid Claude Code conflict

### v1.0.0 — February 28, 2026

- Initial release with enhanced frontend-design skill
- 17 design commands: /polish, /audit, /distill, /bolder, and more
- Support for Cursor, Claude Code, Gemini CLI, and Codex CLI
- Interactive command cheatsheet

## 07 — FAQ

**Where do I put the downloaded files?**
The easiest way is `npx skills add pbakaus/impeccable`, which auto-detects your AI harness and places files correctly. If you downloaded the universal ZIP, extract it to your project root. It creates hidden folders for each supported tool: `.cursor/`, `.claude/`, `.gemini/`, `.codex/`, `.agents/`, and `.github/`. Project-level installation takes precedence and lets you version control your skills.

**How do I update to the latest version?**
Run `npx impeccable skills update` from your project root. Alternative: `npx skills add pbakaus/impeccable` re-installs from scratch. Claude Code plugin: open `/plugin`, go to Discover tab. Your `PRODUCT.md` and `DESIGN.md` context files are never overwritten.

**I used to type `/critique` directly. How do I get that back?**
Pinning is built in. Run `/impeccable pin critique` and `/critique` becomes a standalone shortcut again. Under the hood it writes a lightweight redirect skill that delegates to `/impeccable critique`, so updates to the parent skill flow through automatically. Examples: `/impeccable pin polish` → `/polish` works again. To remove: `/impeccable unpin critique`.

**Commands or skills aren't appearing. What do I do?**
For commands: type `/impeccable` in your AI harness. For skills: skills are applied automatically when relevant; to verify, explicitly mention "use the impeccable skill" in your prompt.

Tool-specific setup:

- **Cursor**: Requires Nightly channel + Agent Skills enabled in Settings → Rules
- **Gemini CLI**: Requires `@google/gemini-cli@preview` + Skills enabled via `/settings`
- **Codex**: Skills do not appear in the normal `/` command picker. Open `/skills` or type `$`. Repo installs live in `.agents/skills/`, user installs in `~/.agents/skills/`. Restart Codex if a new skill does not show up.

**Is Impeccable free?**
Yes. Everything is **Apache 2.0**: skills, commands, CLI, and the detection engine. Fully open source, free for everyone.

## Work with me

Impeccable is built by **Renaissance Geek** (Paul Bakaus). He works with enterprise teams on large-scale rollouts, custom integrations, and training for designers and developers.
