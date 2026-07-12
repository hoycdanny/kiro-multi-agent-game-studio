---
inclusion: always
---

# Game Design Document (GDD)

> **這份檔案是什麼**：GDD 全名 **Game Design Document（遊戲設計文件）**，是本專案「遊戲怎麼設計」的**單一真相來源（Single Source of Truth）**——遊戲概念、核心循環、系統規格、數值平衡都寫在這裡。所有 Agent（設計 / 美術 / 引擎 / QA）動工前都會先讀它，確保大家依同一份設計走，不會各做各的。
>
> 本框架假設你**專注開發一款遊戲**（我們提供的是多種開發顧問與多種引擎供你選）；所以設計文件就這一份，放在 `.kiro/steering/project/`，`inclusion: always` 每次對話自動載入。
>
> **維護者**：`design/game-designer`，各遊戲類型的 Domain Expert（如 `slot-game-expert`、`rpg-systems-expert`、`shooter-expert`…）把各自的規格整合進來。目前為骨架，各章節由對話中逐步填寫，**不要自行捏造內容**。

## 遊戲名稱
_（尚未定義）_

## 遊戲概念
_（尚未定義）_

## 核心循環（Core Loop）
_（尚未定義）_

## 目標平台
_（尚未定義，對照 root README「待確認事項」）_

## 系統規格
_（各系統的規格會以子章節或連結形式加入）_

## 數值平衡表
_（尚未定義）_

## Postmortem（階段回顧）

> 每個 milestone（見 `.kiro/steering/project/milestones.md`）完成後，建議在這裡補一則回顧，記錄「這階段學到什麼」，避免下一階段重複同樣的判斷失誤。**目前沒有自動觸發機制**——沒有工具能偵測「milestone 已完成」，需要使用者或 `producer` 在確認階段完成時主動提出要不要寫。誠實聲明：這是流程建議，不是本框架已自動化的能力。

### Postmortem 範本

```markdown
### [Milestone 名稱] Postmortem — YYYY-MM-DD

**做得好的地方**：
-

**遇到的問題**：
-

**下階段要調整的做法**：
-

**是否有 agent 分工或 Contract 流程需要調整**：
-
```

（尚未有任何階段完成，此區塊目前為空）

## 變更紀錄

| 日期 | 變更 | 負責人 |
|------|------|--------|
| - | 建立骨架 | game-designer |
