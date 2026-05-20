---
url: "https://openspec.dev/"
clipped: 2026-05-20
title: "OpenSpec — A lightweight spec-driven framework"
publisher: "Fission AI"
---

# OpenSpec — A lightweight spec-driven framework

A lightweight spec-driven framework

- Universal
- Open Source
- No API Keys
- No MCP

## Get Started

```bash
npm install -g @fission-ai/openspec@latest
```

- GitHub: https://github.com/Fission-AI/OpenSpec/
- Discord: https://discord.gg/YctCnvvshC

## Supported Tools

### Native support

These tools have native OpenSpec integration with custom slash commands built-in.

- Claude Code
- Cursor
- Codex
- GitHub Copilot
- OpenCode
- Windsurf
- Gemini CLI
- Antigravity
- Cline
- RooCode
- Kilo Code
- Amazon Q
- Qoder
- Auggie CLI
- Qwen Code
- CodeBuddy
- CoStrict
- Crush
- Factory Droid
- iFlow
- (+ 16 more)

## Features

### 1. Review intent, not just code

Each OpenSpec change produces a **spec delta** that captures the change in requirements of the system. This makes it easy for developers to understand how they're modifying the system and what will need to change. It also allows reviewers to understand the change itself without having to dig through the code and quickly gain a high level understanding.

Example delta in `openspec/specs/auth-session/spec.md`:

```diff
### Requirement: Session expiration
- The system SHALL expire sessions after a configured duration.
+ The system SHALL support configurable session expiration periods.

#### Scenario: Default session timeout
- GIVEN a user has authenticated
- - WHEN 24 hours pass without activity
+ - WHEN 24 hours pass without "Remember me"
- THEN invalidate the session token

+ #### Scenario: Extended session with remember me
+ - GIVEN user checks "Remember me" at login
+ - WHEN 30 days have passed
+ - THEN invalidate the session token
+ - AND clear the persistent cookie
```

### 2. Context that persists

Your specs live in the repository alongside your code, organized by capability. When an agent needs context about how a feature should work, it reads the spec. When a new developer joins, they can browse the library to understand the system. Context doesn't disappear when a chat session ends or someone leaves the team.

```
$ tree openspec/specs/
openspec/specs/
├── auth-login/
│   └── spec.md
├── auth-session/
│   └── spec.md
├── checkout-cart/
│   └── spec.md
└── checkout-payment/
    └── spec.md
```

Example `openspec/specs/auth-session/spec.md`:

```markdown
# auth-session Specification

## Purpose
Manage user session lifecycle including creation, validation, and expiration.

## Requirements
### Requirement: Session expiration
The system SHALL expire sessions after a configured duration.

#### Scenario: Default session timeout
- GIVEN a user has authenticated
- WHEN 24 hours pass without activity
- THEN invalidate the session token
- AND require re-authentication
```

### 3. Something to review in seconds

When you describe a change you want to make, OpenSpec generates everything needed to review it: a proposal document, broken-down implementation tasks, technical design decisions, and the spec deltas showing how requirements will change. You review and refine the plan before any code is written, catching misalignment early.

Example workflow:

```
> /openspec:proposal Add remember me checkbox with 30-day sessions

Searching existing specs for authentication requirements...
Read(openspec/specs/auth-session/spec.md)
Searching existing codebase for session handling...
Read(src/auth/session.ts)
Creating proposal and breaking down implementation tasks...

Created change proposal with ID add-remember-me

openspec/changes/add-remember-me/
├── proposal.md   ← describe the change
├── design.md     ← technical decisions
├── tasks.md      ← implementation tasks
└── specs/        ← spec deltas
    └── auth-session/
        └── spec.md

This change affects 1 spec, with 3 phases and 8 tasks
```

## Coming Soon: Workspaces

In Development — OpenSpec has become the go-to planning layer for many developers. Now building for teams.

Problems being tackled:

- Large codebases
- Multi-repo planning
- Customization and integrations
- Better collaboration

## Frequently Asked Questions

### How is OpenSpec different from my agent's built-in plan mode?

Plan mode is great for a single chat session. OpenSpec focuses on plans that extend over multiple sessions, or that you want to share with others. A workspace for feature planning lets you plan better and refine as you go. It's something you bring through the entire development lifecycle, not just one conversation.

### What makes OpenSpec different from other planning tools?

1. **Lightweight** — Minimal steps, minimal process. Get building as quickly as possible.
2. **Brownfield-first** — Most tools assume you're starting fresh. OpenSpec focuses on mature codebases where the real struggle is figuring out how the current system works.
3. **Specs live in your code** — Other tools only use requirements during planning, then throw them away. OpenSpec preserves the functional requirements behind your code as living documentation.

### Why use a spec instead of just writing a detailed prompt?

Specs serve as alignment. A way to structure your thinking in a single space before a single line of code is written. Better clarity on what you're building, and better context for your agent when executing your plan.

### Can I use OpenSpec on an existing codebase?

Yep! Specs get created as you build. They're exploring generating specs for existing codebases, but the view is that trying to generate all your specs upfront is a waste of time. Create specs as you need them and build your way through.

### What happens when I switch between coding agents?

Goal is to be a universal planning layer you can bring with you anywhere, no matter what coding agent you use. Coding agents are improving rapidly. What's popular this month might not be next month. Your specs shouldn't care.

### Where do specs live?

In your codebase. They should be checked in — they provide visibility into how the system works and the intent it was built with.

### How do teams share and collaborate on specs?

Specs and changes live in your code, so teams collaborate through git — PRs, reviews, the usual workflow. Deeper team features coming for large codebases, multi-repo systems, microservices.

### Wait isn't this just waterfall?

Waterfall fails because of rigid plans and months of upfront planning. This is neither. Get to a good enough plan and start coding — minimal effort, lightweight process. When things change, update the spec and keep going.

### I'm a vibe coder - is this tool for me?

Honestly? It depends. If you're looking for a magic tool that plans everything for you without any effort on your part, this isn't it. Specs only work if you actually read them, think through them, and engage with them. This is a tool to help you build the right thing — but it works best when you meet it halfway.
