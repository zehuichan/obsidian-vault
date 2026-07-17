---
url: "https://github.com/mastra-ai/mastra"
clipped: 2026-07-17
title: "Mastra — TypeScript framework for AI-powered applications and agents"
---

# Mastra

Mastra is a framework for building AI-powered applications and agents with a modern TypeScript stack.

It includes everything you need to go from early prototypes to production-ready applications. Mastra integrates with frontend and backend frameworks like React, Next.js, and Node, or you can deploy it anywhere as a standalone server. It's the easiest way to build, tune, and scale reliable AI products.

**Repository:** https://github.com/mastra-ai/mastra  
**Website:** https://mastra.ai  
**Docs:** https://mastra.ai/docs  
**Stars:** ~26.3k · **Forks:** ~2.4k · **Language:** TypeScript (~99%)  
**Y Combinator:** W25

## Why Mastra?

Purpose-built for TypeScript and designed around established AI patterns, Mastra gives you everything you need to build great AI applications out-of-the-box.

Some highlights include:

- **Model routing** — Connect to 40+ providers through one standard interface. Use models from OpenAI, Anthropic, Gemini, and more. Docs: https://mastra.ai/models

- **Agents** — Build autonomous agents that use LLMs and tools to solve open-ended tasks. Agents reason about goals, decide which tools to use, and iterate internally until the model emits a final answer or an optional stopping condition is met. Docs: https://mastra.ai/docs/agents/overview

- **Workflows** — When you need explicit control over execution, use Mastra's graph-based workflow engine to orchestrate complex multi-step processes. Mastra workflows use an intuitive syntax for control flow (`.then()`, `.branch()`, `.parallel()`). Docs: https://mastra.ai/docs/workflows/overview

- **Human-in-the-loop** — Suspend an agent or workflow and await user input or approval before resuming. Mastra uses storage to remember execution state, so you can pause indefinitely and resume where you left off. Docs: https://mastra.ai/docs/workflows/suspend-and-resume

- **Context management** — Give your agents the right context at the right time. Provide conversation history, retrieve data from your sources (APIs, databases, files), and add human-like memory with Observational Memory so your agents behave coherently.

- **Integrations** — Bundle agents and workflows into existing React, Next.js, or Node.js apps, or ship them as standalone endpoints. When building UIs, integrate with agentic libraries like Vercel's AI SDK UI and CopilotKit to bring your AI assistant to life on the web.

- **MCP servers** — Author Model Context Protocol servers, exposing agents, tools, and other structured resources via the MCP interface. These can then be accessed by any system or agent that supports the protocol. Docs: https://mastra.ai/docs/tools-mcp/mcp-overview

- **Production essentials** — Shipping reliable agents takes ongoing insight, evaluation, and iteration. With built-in evals and observability, Mastra gives you the tools to observe, measure, and refine continuously.

## Get started

The **recommended** way to get started with Mastra is by running the command below:

```shell
npm create mastra@latest
```

Follow the [Installation guide](https://mastra.ai/docs/getting-started/installation) for step-by-step setup with the CLI or a manual install.

If you're new to AI agents, check out [templates](https://mastra.ai/docs/getting-started/templates), [course](https://mastra.ai/course), and [YouTube videos](https://youtube.com/@mastra-ai).

### Alternative: pre-built prompt for coding agents

```md
Make new Mastra project. Mastra = framework for AI apps + agents on modern TypeScript stack. Before run command, ask these questions one by one. Wait for answers unless already given:

Project name? (default: "my-mastra-app")
Provider? (default: "openai", options: "openai", "anthropic", "groq", "google", "cerebras", "mistral")
Provider rules:

Allowed provider -> use it.
Any other value -> use "openai".
Run with answers: npm create mastra@latest <project-name> -- --default --llm <provider>

After project created, go to project dir. Start dev server: npx bgproc start -n <project-name> -w -- npm run dev

Start Mastra Studio at http://localhost:4111. Studio = UI for build, test, manage agents, workflows, tools.

Also tell: Mastra model router give 3000+ models from many providers: https://mastra.ai/models
```

Mastra Studio (dev UI) runs at `http://localhost:4111` after starting the project.

## Documentation

- Official docs: https://mastra.ai/docs
- Build with AI guide: https://mastra.ai/docs/getting-started/build-with-ai

## Monorepo layout (high level)

Key top-level areas in the repository:

- `packages/` — core packages
- `docs/` — documentation
- `examples/`, `templates/` — examples and starter templates
- `client-sdks/`, `agent-sdks/` — client / agent SDKs
- `server-adapters/`, `deployers/` — server and deploy integrations
- `stores/`, `workflows/`, `voice/`, `observability/` — storage, workflows, voice, observability
- `integrations/`, `channels/`, `browser/`, `auth/` — integrations and channels
- `ee/` — enterprise-licensed (source-available) code
- `mastracode/` — Mastra Code related tooling

## Licensing

This repository uses a dual-license model:

- **Apache License 2.0** — The core framework and the vast majority of this codebase is open source under Apache-2.0.
- **Mastra Enterprise License** — Code in any directory named `ee/` (e.g., `packages/core/src/auth/ee/`) is source-available under the Mastra Enterprise License. These features require a valid enterprise license for production use but can be freely used for development and testing.

See LICENSE.md for the full license mapping and ee/LICENSE for the enterprise license terms.

## Security

Responsible disclosure: security@mastra.ai

## Topics

nodejs, javascript, typescript, ai, reactjs, mcp, nextjs, tts, chatbots, workflows, agents, llm, evals

## Support

- Discord: https://discord.gg/BTYqqHKUrf
- Twitter/X: https://x.com/mastra
