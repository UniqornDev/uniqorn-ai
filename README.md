# Using AI with Uniqorn

[Uniqorn](https://uniqorn.dev) is a serverless platform for Java REST APIs. You write code, `git push`, and it's live! No containers, no pipelines, no infrastructure to manage.

This repo helps you use AI to build, iterate on, and integrate with Uniqorn endpoints.

## What is Uniqorn?

Uniqorn takes the most common thing developers build: REST APIs, and strips away everything that isn't the actual logic.

- **Write. Push. Live.** Uniqorn includes a built-in Git server. Write a Java endpoint, push it, and it compiles, deploys, and serves requests instantly. The push *is* the deployment.
- **Isolated by the runtime, guided by the API.** Every instance is its own JVM in its own container, so your code is separated from every other tenant by the runtime itself rather than by a policy check. On top of that, the hosted trial and personal plans keep endpoints inside the framework API: on deploy, Uniqorn reads the compiled bytecode and rejects types like `java.io.File` or `java.net.Socket` with `422 Use of restricted type`. That restriction is a guardrail that keeps entry-level code in the intended model, not a security boundary, and team, enterprise, and self-hosted instances do not have it.
- **Every version, always.** Every push creates a version. Any version can be restored instantly.

No Kubernetes. No Docker. No CI/CD pipeline. No YAML files. Just your code.

Learn more at [uniqorn.dev](https://uniqorn.dev).

## What's in this repo

This repo provides four resources for using AI with Uniqorn, depending on your workflow.

### [`CONTEXT.md`](CONTEXT.md) Start here

The *why and the shape*: what Uniqorn is, the philosophy that predicts what it will and won't
do, the non-goals, the repo layout, and the honest boundaries (what the deploy-time type
restriction actually is, and what it is not). `BACKEND.md` teaches an AI the API; this teaches it
the judgement — which is what keeps it from inventing patterns the platform deliberately
refuses.

Pair it with `BACKEND.md` for any non-trivial work: together they are under 8k tokens, small
enough to sit in context permanently.

### [`BACKEND.md`](BACKEND.md) For building the API

A structured instruction file that teaches AI agents how to write Uniqorn endpoints. Use it in two ways:

**As a dedicated agent** — create a specialized "Uniqorn developer" agent that knows the platform inside out:

- **Claude Code** — save as `.claude/agents/uniqorn.md` to get a `/uniqorn` agent you can invoke directly
- **Cursor** — add as a custom agent in your workspace settings
- **OpenAI GPTs** — use as the system instructions for a custom GPT
- Any platform that supports custom agent definitions

**As project-level instructions** — include it as context so your general-purpose AI assistant understands Uniqorn when working in your repo:

- **Claude Code** — reference from your `CLAUDE.md` or use directly as `CLAUDE.md`
- **Cursor** — use as `.cursorrules`
- **GitHub Copilot** — use as `.github/copilot-instructions.md`
- **Windsurf** — use as `.windsurfrules`

**For chat-based AI** (ChatGPT, Claude, Gemini, etc.), no tooling setup required: paste the contents of `BACKEND.md` into the chat, prefixed with

> You are an AI agent with the following definition:

then start describing what you want to build. Ideal for prototyping an endpoint in conversation, understanding the API patterns, or generating code you'll push manually via Git.

### [`FRONTEND.md`](FRONTEND.md) For building the user interface

The companion to `BACKEND.md`, on the browser side. Uniqorn instances serve the Aeonics Frontend
Framework at `/ae/`: a vanilla, build-free ES-module toolkit — hash router, `Page` class, DOM
builder, `fetch` wrapper, modals, toasts, translations. This file teaches an AI how to build a
single page application with it in `www/` and wire it to your endpoints.

Install it exactly like `BACKEND.md` — as a dedicated agent (`.claude/agents/aff.md`), as project
instructions, or pasted into a chat. Use it alongside `BACKEND.md` when a feature spans both
sides.

### [`MCP.md`](MCP.md) For connecting AI agents to your live APIs

Uniqorn natively supports the [Model Context Protocol (MCP)](https://modelcontextprotocol.io), which means your deployed endpoints can be exposed as tools that AI agents call directly.

This guide covers:
- Wiring up Claude Desktop, Claude Code, or other MCP clients to your Uniqorn instance
- Practical examples of AI agents that use your live APIs

This is where it gets interesting: you use AI to *build* the endpoint, then AI agents *use* the endpoint, all on the same platform.
