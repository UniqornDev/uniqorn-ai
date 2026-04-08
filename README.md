# Using AI with Uniqorn

[Uniqorn](https://uniqorn.dev) is a serverless platform for Java REST APIs. You write code, `git push`, and it's live! No containers, no pipelines, no infrastructure to manage.

This repo helps you use AI to build, iterate on, and integrate with Uniqorn endpoints.

## What is Uniqorn?

Uniqorn takes the most common thing developers build: REST APIs, and strips away everything that isn't the actual logic.

- **Write. Push. Live.** Uniqorn includes a built-in Git server. Write a Java endpoint, push it, and it compiles, deploys, and serves requests instantly. The push *is* the deployment.
- **Safe by construction.** Your code runs in a sandbox. It can handle requests, query databases, and return responses. It cannot touch the filesystem, open sockets, or reach into system internals. Dangerous operations are blocked at compile time.
- **Every version, always.** Every push creates a version. Any version can be restored instantly.

No Kubernetes. No Docker. No CI/CD pipeline. No YAML files. Just your code.

Learn more at [uniqorn.dev](https://uniqorn.dev).

## What's in this repo

This repo provides three resources for using AI with Uniqorn, depending on your workflow.

### [`AGENT.md`](AGENT.md) For AI agents

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

### [`PROMPT.md`](PROMPT.md) For chat-based AI

A ready-to-use prompt you can paste into any chat-style AI (ChatGPT, Claude, Gemini, etc.) to get help writing Uniqorn endpoints. No tooling setup required: just copy, paste, and start describing what you want to build.

Ideal when you want to:
- Quickly prototype an endpoint in conversation
- Get help understanding Uniqorn's API patterns
- Generate code you'll push manually via Git

### [`MCP.md`](MCP.md) For connecting AI agents to your live APIs

Uniqorn natively supports the [Model Context Protocol (MCP)](https://modelcontextprotocol.io), which means your deployed endpoints can be exposed as tools that AI agents call directly.

This guide covers:
- Wiring up Claude Desktop, Claude Code, or other MCP clients to your Uniqorn instance
- Practical examples of AI agents that use your live APIs

This is where it gets interesting: you use AI to *build* the endpoint, then AI agents *use* the endpoint, all on the same platform.
