# 範例：JWT 驗證任務的完整交接流程

示範一個任務從開始到結束，經過 3 個 provider/帳號、3 台主機、4 個角色的交接過程。

- [steps.md](steps.md)：每一棒的 `state.md` 快照與 commit 訊息，依時間順序排列。
- [docs/handoff/](docs/handoff/)：各角色的產出文件範本。
- [roles/implementer/memory.md](roles/implementer/memory.md)：角色記憶範例。

> 這些都是示範內容，不是真的專案。給 agent 讀的部分用英文，說明文字用中文。

## 角色與人員

| 角色 | 這次由誰扮演 |
|---|---|
| `planner` | `claude-opus-5-5@claude-a/host-1` |
| `reviewer` | `gpt-6-astra@codex/host-2` |
| `implementer` | `claude-sonnet-5-5@claude-a/host-3`，quota 用完後換成 `@claude-b/host-3` |
| `auditor` | `gpt-6-astra@codex/host-2`（選配） |

## 流程總覽

| 步驟 | 誰 | 做什麼 | Relay-To | 示範重點 |
|---|---|---|---|---|
| 00 | user | 開任務、寫 Pipeline | — | 任務起點 |
| 01 | planner | 寫 plan | reviewer | 一般交棒 |
| 02 | reviewer | 審 plan | planner | 換 provider、換機器 |
| 03 | planner | 處理意見、寫 spec | implementer | 逐條處理審查意見 |
| 04 | implementer | 實作第一步 | （無） | checkpoint 不寫 Relay |
| 05 | implementer | 換帳號接手 | （無） | quota 用完，同一角色換人，不算交棒 |
| 06 | implementer | 發現 spec 有問題 | planner | 退回（detour），不自己猜 |
| 07 | planner | 回答問題 | implementer | detour：回到 `Return-To`，`Step` 不推進 |
| 08 | implementer | 完成實作 | planner | 交出結果與證據 |
| 09 | planner | 驗收 | user | 自己跑驗證，不只看報告 |
| 10 | auditor | 第三方審計 | user | Pipeline 外的角色不動 `state.md`；獨立判斷，直接交給 user |
| 11 | user | 記住某件事 | （無） | 寫進角色記憶 |

## 查歷史

```bash
git log -p .handoff/state.md                                   # 每一棒的 state
git log --format='%h %ad %(trailers:key=Relay-To,valueonly)' --date=short  # 誰交給誰
```
