<!-- Rendered at https://github.com/cats-inc -->

**English** · [繁體中文](https://github.com/cats-inc/.github/blob/main/profile/README.zh-TW.md)

# Cats Inc

Cats Inc is a project brand maintained by the individual developer [sammykenny2](https://github.com/sammykenny2), not a registered company; all Cats repositories are MIT licensed.

> The monsters ran a power company. The cats run a compute one.

An open-source **AI agent runtime** that runs the agent CLIs, model APIs and local
models you already have behind one interface — and a **multi-agent collaboration
platform** that puts them to work. You install and sign in to each provider yourself,
and each provider's terms decide what your plan allows.

Most agent tooling is either a thin wrapper around one vendor's CLI or a chat window
with a model key pasted into it. This is the other shape: execution lives in one place
and is shared, and agents are long-lived participants rather than one-off completions.

Three repositories, one stack:

```bash
npx @cats-inc/cats-one
```

That single command starts the runtime, waits for it to become healthy, then starts the
platform on top of it.

---

## The stack

```
        ┌──────────────────────────────────────────────────────┐
        │  cats-one                                            │
        │  one-command bootstrap — starts the runtime, waits   │
        │  on /health, then launches the platform              │
        └───────────────────────┬──────────────────────────────┘
                                │
             ┌──────────────────┴───────────────────┐
             ▼                                      ▼
   ┌──────────────────────┐   HTTP / SSE   ┌──────────────────────┐
   │  cats-platform       │ ─────────────▶ │  cats-runtime        │
   │  collaboration layer │                │  execution boundary  │
   │  React · Vite ·      │                │  Node · Hono         │
   │  Electron            │                │                      │
   └──────────────────────┘                └──────────┬───────────┘
                                                      │
                        ┌─────────────────────────────┼─────────────────────────┐
                        ▼                             ▼                         ▼
                   agent CLIs                    model APIs               local models
              16 provider families          Claude · Codex · Gemini          Ollama
```

The boundary between the two halves is deliberate. `cats-platform` never spawns a
provider process or holds a provider credential — it asks `cats-runtime` to. That keeps
provider sprawl in one place and lets the product layer be rewritten without touching
execution.

---

## Repositories

| Repository | What it is | Package |
| --- | --- | --- |
| **[cats-runtime](https://github.com/cats-inc/cats-runtime)** | The execution boundary. Sessions, streaming, workspace isolation, tools, protocols, metering. | [![npm](https://img.shields.io/npm/v/@cats-inc/cats-runtime?label=%40cats-inc%2Fcats-runtime)](https://www.npmjs.com/package/@cats-inc/cats-runtime) |
| **[cats-platform](https://github.com/cats-inc/cats-platform)** | The collaboration layer. Chat-first multi-agent workspace, orchestration, approvals, desktop app. | [![npm](https://img.shields.io/npm/v/@cats-inc/cats-platform?label=%40cats-inc%2Fcats-platform)](https://www.npmjs.com/package/@cats-inc/cats-platform) |
| **[cats-one](https://github.com/cats-inc/cats-one)** | The launcher. Boots both of the above in one shot. | [![npm](https://img.shields.io/npm/v/@cats-inc/cats-one?label=%40cats-inc%2Fcats-one)](https://www.npmjs.com/package/@cats-inc/cats-one) |

All three are MIT licensed and published to npm.

---

## cats-runtime — the execution boundary

A single HTTP surface in front of everything that can run an agent turn.

- **One session model across very different backends.** Subscription agent CLIs, hosted
  model APIs, and local models all create, stream, cancel, reset, fork, and delete
  through the same lifecycle primitives.
- **16 CLI provider families** — Claude, Codex, Antigravity, Cursor, Copilot, OpenCode,
  Kilo, Goose, Pi, Auggie, Junie, Kiro, Grok, Cline, Devin, and Aider — plus API-backed
  Claude / Codex / Gemini families and local Ollama.
- **Workspace isolation** backed by git worktrees, with deterministic
  prepare / recreate / cleanup semantics and explicit discard / merge / preserve
  policies on reset and delete.
- **Streaming** over SSE or NDJSON, with runtime-owned `content_block` projections so
  hosts can render transcripts without knowing which provider produced them.
- **Runtime-hosted tools** (`list_files`, `read_file`, `write_file`, `grep`, `run_shell`)
  for backends that don't bring their own.
- **Capability truth, not hard-coded assumptions.** `/providers/config` reports each
  provider's normalized text / tool / progress / block posture, so hosts branch on
  declared capability instead of on provider name.
- **Usage metering and guardrails** with warn / block / cooldown flows and incident
  surfacing.
- **Skills** — a family-aware library with backend-aware delivery modes
  (`filesystem`, `instructions`, `none`) and per-target re-derivation.

### Protocols

| Protocol | Status |
| --- | --- |
| **MCP** | Served. Authoritative execution on `POST /mcp`, plus a published `cats-runtime mcp` stdio proxy and curated mutation tools. |
| **ACP** | Served. Bounded facade on `POST /acp` and a direct stdio carrier for IDE and client integration; plus provider-side ACP across 13 of the CLI provider families. |
| **A2A** | In progress. Peer routing hints, peer diagnostics, and a policy-gated peer execution route exist; the public agent-card and JSON-RPC surface are not published yet. |

---

## cats-platform — the collaboration layer

A chat-first workspace where agents are named, persistent collaborators.

- **Orchestration.** A global orchestrator plus direct-agent routing, deterministic
  `@mention` handling, visible presence states, and machine-readable room-routing state.
- **An operator loop next to the transcript** — pending approvals, progress, activity,
  traces, run inspection, and approve / reroute / retry / acknowledge seams.
- **Contract-first planning** with approval-gated dispatch and checkpoint-driven
  multi-step execution plans with recovery actions.
- **Cats Work** and **Cats Code** dashboards over a shared task schema, project and
  work-item detail, artifacts, activity, and timelines.
- **A real desktop app.** Electron host that supervises local runtime and platform
  processes, produces a Windows NSIS installer, stages cross-platform packaging, and
  owns tray/background lifecycle plus an update path.
- **Packaged setup and recovery.** First-run provider scan, resumable setup recovery,
  and cross-layer bootstrap diagnostics that stitch runtime, product, and host state
  into one chronology.

---

## Tech stack

**TypeScript** end to end, on **Node 22+**.

| Layer | Choices |
| --- | --- |
| Runtime service | Hono, native SSE/NDJSON streaming, YAML-backed provider topology |
| Platform server | Node, file-backed state and transcript persistence |
| Renderer | React, React Router, TanStack Query, Vite, Tailwind |
| Desktop | Electron, electron-builder, NSIS, electron-updater |
| Build & test | esbuild, tsx, Vitest, Testing Library |

---

## Repository at a glance

| | cats-runtime | cats-platform | cats-one |
| --- | ---: | ---: | ---: |
| Commits | ~980 | ~4,000 | ~20 |
| Tracked files | ~730 | ~2,740 | 12 |
| Test files | 193 | 395 | — |
| Docs (markdown) | 187 | 417 | 3 |

Both main repositories carry `ROADMAP.md`, `PROGRESS.md`, architecture decision records
under `docs/decisions/`, and written plans under `docs/plans/`. Design intent is
committed alongside the code rather than reconstructed after the fact.

---

## Where to start reading

Curious about the execution model? → [`cats-runtime/docs/architecture.md`](https://github.com/cats-inc/cats-runtime/blob/main/docs/architecture.md)

Curious about the collaboration model? → [`cats-platform/README.md`](https://github.com/cats-inc/cats-platform#readme)

Curious how the decisions were made? → `docs/decisions/` in either repository

Just want to run it? → `npx @cats-inc/cats-one`

---

## License

MIT, across all repositories.

---

*The staff: QQ, 奶奶, and 財財. In memory of 將將 (2008 – 5 November 2023) and my beloved 醜醜 (2010 – 24 September 2025).*
