[English](https://github.com/cats-inc/.github/blob/main/profile/README.md) · **繁體中文**

# Cats Inc

Cats Inc 是個人開發者 [sammykenny2](https://github.com/sammykenny2) 維護的專案品牌，並非已登記公司；所有 Cats repositories 均採 MIT 授權。

> 怪獸開的是電力公司，醜貓開的是算力公司。

Cats 是一個仍在開發中的個人專案，用來學習如何讓 AI agent 實際參與工作。目標是把你已經
安裝好的 agent CLI、model API 與本機模型放在同一個 runtime 後面執行，並在其上做一個
聊天形式的工作區，讓多個 agent 可以一起協作。各家 provider 都由你自行安裝與登入，
你的方案能怎麼用，以各家 provider 的條款為準。

這是一個實驗性質的專案，不是完成的產品。已有部分功能實作，仍在測試與驗證中，詳見[目前狀態](#目前狀態)。

---

## 開發方式

大部分程式碼是以 AI 輔助開發的方式完成，搭配 Codex、Claude Code、Antigravity 等 coding agent；
由我負責整理需求、檢視設計方向與測試結果。每個 repo 都保留 `ROADMAP.md`、`PROGRESS.md`，
以及 `docs/` 下的設計決策與計畫，讓設計意圖與目前進度和程式碼放在一起。

---

## 目前狀態

| 項目 | 狀態 |
| --- | --- |
| 在同一個介面後面執行多種 agent CLI、model API 與本機模型（Ollama） | 完成度最高 |
| Session 生命週期：建立、串流、取消、重設、刪除 | 完成度最高 |
| 工作區隔離、用量計量、MCP 與 ACP 介面 | 部分完成：已有程式碼與自動化測試，實際操作測試很少 |
| 聊天工作區、`@mention` 路由、審核、執行追蹤、桌面應用 | 部分完成：初期版本，部分功能尚未實際測試 |
| 多 agent 協調、長期記憶與檢索、A2A | 尚無可用版本 |

各 repo 的 `PROGRESS.md` 有逐項的詳細進度。

---

## 架構

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

`cats-platform` 不會自己啟動 provider process，也不持有 provider 憑證，一律請 `cats-runtime` 代勞。

---

## Repositories

| Repository | 這是什麼 | 套件 |
| --- | --- | --- |
| **[cats-runtime](https://github.com/cats-inc/cats-runtime)** | 在同一個 HTTP 介面後面執行 agent CLI、model API 與本機模型。 | [![npm](https://img.shields.io/npm/v/@cats-inc/cats-runtime?label=%40cats-inc%2Fcats-runtime)](https://www.npmjs.com/package/@cats-inc/cats-runtime) |
| **[cats-platform](https://github.com/cats-inc/cats-platform)** | 建立在 runtime 之上的聊天工作區與桌面應用。 | [![npm](https://img.shields.io/npm/v/@cats-inc/cats-platform?label=%40cats-inc%2Fcats-platform)](https://www.npmjs.com/package/@cats-inc/cats-platform) |
| **[cats-one](https://github.com/cats-inc/cats-one)** | 一行指令同時啟動 runtime 與平台的啟動器。 | [![npm](https://img.shields.io/npm/v/@cats-inc/cats-one?label=%40cats-inc%2Fcats-one)](https://www.npmjs.com/package/@cats-inc/cats-one) |
| **[cats-apps](https://github.com/cats-inc/cats-apps)** | 桌面應用的小型附加 App，第一個是用量儀表板。 | 早期預覽 |
| **[cats-plugins](https://github.com/cats-inc/cats-plugins)** | 把第三方 skills 打包給 Cats 使用的實驗。 | 早期預覽 |

使用 TypeScript、Node 22+ 開發；runtime 服務採用 Hono，工作區與桌面應用採用 React、Vite 與 Electron。

---

## 試用

```bash
npx @cats-inc/cats-one
```

這行指令會先啟動 runtime，等它進入 healthy 狀態後再啟動平台。目前仍有不少粗糙之處，
讓任何 agent 修改檔案之前，請先備份重要資料。

---

## 授權

全部 repo 採 MIT 授權。

---

*本公司員工：QQ、奶奶、財財。謹紀念將將（2008 ～ 2023.11.5）與摯愛醜醜（2010 ～ 2025.9.24）。*
