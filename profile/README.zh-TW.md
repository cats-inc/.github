[English](https://github.com/cats-inc/.github/blob/main/profile/README.md) · **繁體中文**

# Cats Inc

> 怪獸開的是電力公司，醜貓開的是算力公司。

一套開源的 **AI agent runtime**，把觸手可及的每一種 agent CLI、model API 與本機模型
都收攏成可用的算力；以及一個把這些算力派上用場的**多 agent 協作平台**。

現在的 agent 工具多半只有兩種形狀：包住單一廠商 CLI 的薄殼，或是貼上一把 model key 的
聊天視窗。這裡走的是第三種 —— 執行收在同一個地方並且共用，agent 是長期存在的參與者，
而不是一次性的 completion。

三個 repo，一套系統：

```bash
npx @cats-inc/cats-one
```

這一行指令會啟動 runtime，等它進入 healthy 狀態，再把平台接上去。

---

## 架構

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

兩層之間的邊界是刻意畫出來的。`cats-platform` 從不自己拉起 provider process，也不持有
provider 憑證 —— 它一律請 `cats-runtime` 代勞。這讓 provider 的複雜度收斂在同一個地方，
產品層也可以整層改寫而不動到執行路徑。

---

## Repositories

| Repository | 這是什麼 | 套件 |
| --- | --- | --- |
| **[cats-runtime](https://github.com/cats-inc/cats-runtime)** | 執行邊界。session、串流、工作區隔離、工具、協定、用量計費。 | [![npm](https://img.shields.io/npm/v/@cats-inc/cats-runtime?label=%40cats-inc%2Fcats-runtime)](https://www.npmjs.com/package/@cats-inc/cats-runtime) |
| **[cats-platform](https://github.com/cats-inc/cats-platform)** | 協作層。聊天為核心的多 agent 工作區、編排、審核、桌面應用。 | [![npm](https://img.shields.io/npm/v/@cats-inc/cats-platform?label=%40cats-inc%2Fcats-platform)](https://www.npmjs.com/package/@cats-inc/cats-platform) |
| **[cats-one](https://github.com/cats-inc/cats-one)** | 啟動器。一次把上面兩個帶起來。 | [![npm](https://img.shields.io/npm/v/@cats-inc/cats-one?label=%40cats-inc%2Fcats-one)](https://www.npmjs.com/package/@cats-inc/cats-one) |

三個 repo 都採 MIT 授權，並已發佈至 npm。

---

## cats-runtime —— 執行邊界

把所有「能跑出一輪 agent turn」的東西，收在同一個 HTTP 介面之後。

- **同一套 session 模型，跨越差異極大的 backend。** 訂閱制 agent CLI、雲端 model API、
  本機模型，都透過同一組生命週期原語完成建立、串流、取消、重設、分支與刪除。
- **16 個 CLI provider 家族** —— Claude、Codex、Antigravity、Cursor、Copilot、OpenCode、
  Kilo、Goose、Pi、Auggie、Junie、Kiro、Grok、Cline、Devin、Aider —— 另有 API backend 的
  Claude / Codex / Gemini 家族與本機 Ollama。
- **以 git worktree 為底的工作區隔離**，具備確定性的 prepare / recreate / cleanup 語意，
  重設與刪除時可明確選擇 discard / merge / preserve 政策。
- **串流** 支援 SSE 或 NDJSON，並由 runtime 產出 `content_block` 投影，讓 host 不必知道
  訊息來自哪個 provider 就能渲染 transcript。
- **runtime 自帶工具**（`list_files`、`read_file`、`write_file`、`grep`、`run_shell`），
  補上那些本身不帶工具的 backend。
- **以能力為準，而非寫死判斷。** `/providers/config` 會回報每個 provider 正規化後的
  text / tool / progress / block 姿態，host 依宣告出來的能力分支，而不是依 provider 名稱。
- **用量計費與護欄**，具備 warn / block / cooldown 流程與事件揭露。
- **Skills** —— 家族感知的技能庫，支援 backend 感知的投遞模式（`filesystem`、
  `instructions`、`none`），並依目標逐次重新推導。

### 協定

| 協定 | 狀態 |
| --- | --- |
| **MCP** | 已服務。`POST /mcp` 為權威執行點，另發佈 `cats-runtime mcp` stdio proxy 與精選的 mutation 工具。 |
| **ACP** | 已服務。`POST /acp` 提供受限 facade，另有直接的 stdio carrier 供 IDE 與 client 整合；另有 provider 側 ACP 涵蓋其中 13 個 CLI provider 家族。 |
| **A2A** | 進行中。peer routing hint、peer 診斷、以及受政策控管的 peer 執行路由皆已存在；公開的 agent-card 與 JSON-RPC 介面尚未發佈。 |

---

## cats-platform —— 協作層

一個以聊天為核心的工作區，agent 在其中是有名字、會延續的協作者。

- **編排。** 全域 orchestrator 搭配直接指派路由、確定性的 `@mention` 處理、可見的在場
  狀態，以及機器可讀的房間路由狀態。
- **緊鄰 transcript 的操作迴圈** —— 待審核項目、進度、活動、trace、run 檢視，以及
  approve / reroute / retry / acknowledge 的操作接縫。
- **契約先行的規劃**，配合審核閘門的派工，以及具備復原動作、由檢查點驅動的多步執行計畫。
- **Cats Work** 與 **Cats Code** 儀表板，建立在共用的 task schema 之上，涵蓋專案與工作項目
  細節、產出物、活動與時間軸。
- **真正的桌面應用。** Electron host 監管本機 runtime 與平台 process，可產出 Windows NSIS
  安裝檔、備妥跨平台封裝產物，並掌管 tray／背景生命週期與更新路徑。
- **封裝後的安裝與復原。** 首次啟動的 provider 掃描、可續行的安裝復原，以及跨層的 bootstrap
  診斷 —— 將 runtime、產品與 host 三邊狀態縫成同一條時序。

---

## 技術棧

全程 **TypeScript**，執行於 **Node 22+**。

| 層 | 選用 |
| --- | --- |
| Runtime 服務 | Hono、原生 SSE/NDJSON 串流、YAML 描述的 provider 拓樸 |
| 平台伺服器 | Node、以檔案為底的狀態與 transcript 持久化 |
| Renderer | React、React Router、TanStack Query、Vite、Tailwind |
| 桌面 | Electron、electron-builder、NSIS、electron-updater |
| 建置與測試 | esbuild、tsx、Vitest、Testing Library |

---

## Repository 概況

| | cats-runtime | cats-platform | cats-one |
| --- | ---: | ---: | ---: |
| Commit 數 | ~980 | ~4,000 | ~20 |
| 追蹤檔案 | ~730 | ~2,740 | 12 |
| 測試檔 | 193 | 395 | — |
| 文件（markdown） | 187 | 417 | 3 |

兩個主 repo 都帶有 `ROADMAP.md`、`PROGRESS.md`、位於 `docs/decisions/` 的架構決策紀錄，
以及 `docs/plans/` 下的書面計畫。設計意圖與程式碼一起進版控，而不是事後回頭補寫。

---

## 從哪裡開始讀

想了解執行模型？→ [`cats-runtime/docs/architecture.md`](https://github.com/cats-inc/cats-runtime/blob/main/docs/architecture.md)

想了解協作模型？→ [`cats-platform/README.md`](https://github.com/cats-inc/cats-platform#readme)

想知道決策怎麼來的？→ 兩個 repo 的 `docs/decisions/`

只想跑跑看？→ `npx @cats-inc/cats-one`

---

## 授權

全部 repo 採 MIT 授權。

---

*本公司員工：QQ、奶奶、財財。謹紀念將將（2008 ～ 2023.11.5）與摯愛醜醜（2010 ～ 2025.9.24）。*
