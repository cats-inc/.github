<!-- Rendered at https://github.com/cats-inc -->

**English** · [繁體中文](https://github.com/cats-inc/.github/blob/main/profile/README.zh-TW.md)

# Cats Inc

Cats Inc is a project brand maintained by the individual developer [sammykenny2](https://github.com/sammykenny2), not a registered company; all Cats repositories are MIT licensed.

> The monsters ran a power company. The cats run a compute one.

Cats is a personal, work-in-progress project for learning how AI agents can be put
to work. It aims to run the agent CLIs, model APIs and local models you already
have behind one runtime, and to build a chat-style workspace on top of it where
several agents can work together. You install and sign in to each provider
yourself, and each provider's terms decide what your plan allows.

This is an experiment, not a finished product. Some features have been implemented
and are still being tested and validated; see [Status](#status).

---

## How it is built

Most of the code is written through AI-assisted development, with coding agents
such as Codex, Claude Code and Antigravity. I set the requirements and review
design choices and test results. Each repository keeps a `ROADMAP.md`, a
`PROGRESS.md`, and its design decisions and plans under `docs/`, so the intent and
the current state are recorded next to the code.

---

## Status

| Area | Status |
| --- | --- |
| Running multiple agent CLIs, model APIs and local models (Ollama) behind one interface | Most complete |
| Session lifecycle: create, stream, cancel, reset, delete | Most complete |
| Workspace isolation, usage metering, MCP and ACP endpoints | Partly done: code and automated tests exist, little hands-on testing |
| Chat workspace, `@mention` routing, approvals, execution tracking, desktop app | Partly done: early versions, some parts not yet tested in real use |
| Multi-agent coordination, long-term memory and retrieval, A2A | No working version yet |

Each repository's `PROGRESS.md` has the detailed, item-by-item state.

---

## How the pieces fit

```
     ┌──────────────────────────────────────────────────────┐
     │  cats-one                                            │
     │  one-command launcher: starts the runtime, waits on  │
     │  /health, then starts the platform                   │
     └───────────────────────┬──────────────────────────────┘
                             │
          ┌──────────────────┴───────────────────┐
          ▼                                      ▼
┌──────────────────────┐   HTTP / SSE   ┌──────────────────────┐
│  cats-platform       │ ─────────────▶ │  cats-runtime        │
│  chat workspace and  │                │  runs the agents     │
│  desktop app         │                │                      │
└──────────────────────┘                └──────────┬───────────┘
                                                   │
                     ┌─────────────────────────────┼─────────────────────────┐
                     ▼                             ▼                         ▼
                agent CLIs                    model APIs               local models
                                         Claude · Codex · Gemini          Ollama
```

`cats-platform` does not start provider processes or hold provider credentials
itself; it asks `cats-runtime` to do that.

---

## Repositories

| Repository | What it is | Package |
| --- | --- | --- |
| **[cats-runtime](https://github.com/cats-inc/cats-runtime)** | Runs agent CLIs, model APIs and local models behind one HTTP interface. | [![npm](https://img.shields.io/npm/v/@cats-inc/cats-runtime?label=%40cats-inc%2Fcats-runtime)](https://www.npmjs.com/package/@cats-inc/cats-runtime) |
| **[cats-platform](https://github.com/cats-inc/cats-platform)** | Chat-style workspace and desktop app built on top of the runtime. | [![npm](https://img.shields.io/npm/v/@cats-inc/cats-platform?label=%40cats-inc%2Fcats-platform)](https://www.npmjs.com/package/@cats-inc/cats-platform) |
| **[cats-one](https://github.com/cats-inc/cats-one)** | Launcher that starts the runtime and the platform with one command. | [![npm](https://img.shields.io/npm/v/@cats-inc/cats-one?label=%40cats-inc%2Fcats-one)](https://www.npmjs.com/package/@cats-inc/cats-one) |
| **[cats-apps](https://github.com/cats-inc/cats-apps)** | Small add-on apps for the desktop app, starting with a usage dashboard. | Early preview |
| **[cats-plugins](https://github.com/cats-inc/cats-plugins)** | Experiments in packaging third-party skills for Cats. | Early preview |

Written in TypeScript on Node 22+, with Hono for the runtime service and React,
Vite and Electron for the workspace and desktop app.

---

## Try it

```bash
npx @cats-inc/cats-one
```

This starts the runtime, waits until it is healthy, then starts the platform.
Expect rough edges, and back up important files before letting any agent modify
them.

---

## License

MIT, across all repositories.

---

*The staff: QQ, 奶奶, and 財財. In memory of 將將 (2008 – 5 November 2023) and my beloved 醜醜 (2010 – 24 September 2025).*
