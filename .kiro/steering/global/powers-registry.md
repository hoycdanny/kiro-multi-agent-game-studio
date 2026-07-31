---
inclusion: always
---

# Kiro Powers 對照表（Powers Registry）

> **這份檔案是什麼**：本專案分成兩層——**Agent 是組織層**（誰做、何時做、用什麼 Contract 交付給誰），**Kiro Power 是領域知識層**（這個工具／這個領域實際上要怎麼正確做）。這份檔案是兩層之間的**唯一對照表**：告訴每個 agent「你的領域知識該去哪裡讀」。
>
> **為什麼要這樣分**：Power 的知識是對真實工具連線驗證過、且獨立於本專案持續更新的；agent prompt 裡手抄的工具細節會過時（本專案曾因此讓 `unity-team` 使用了數個不存在的 `manage_*` action）。所以**工具與領域知識的單一真相來源在 Power，不在 agent prompt**。
>
> **維護者**：`producer`（跨層對照的維護者）。新增 Power 或新增對應 agent 時，同步更新本表。

## 分層原則（所有 agent 適用）

| 這件事 | 寫在哪 | 誰負責 |
|--------|--------|--------|
| 角色定位、在 Pipeline 的位置、Contract／Delivery Manifest 紀律、與相鄰角色的職責界線 | agent 的 `.md` prompt | 本專案 |
| 工具的實際名稱與參數、任務的正確操作順序、領域最佳實踐、平台限制、範本 | 對應 Power 的 `steering/` 與 `templates/` | Power repo |
| 命名／poly budget／音訊格式等跨團隊產出規範 | `.kiro/steering/global/asset-standards.md` | 本專案 |
| 遊戲設計內容與數值 | `.kiro/steering/project/gdd.md` | 本專案 |

**不要把 Power 的內容抄進 agent prompt**。需要時去讀，不要複製。

## Power 在磁碟上的位置

Powers 是**全機安裝**（跟著使用者的 Kiro，不跟著本 repo 走）：

| 路徑 | 內容 | 什麼時候用 |
|------|------|-----------|
| `~/.kiro/powers/installed/<power>/steering/<檔名>.md` | 已啟用 Power 的領域知識 | **主要來源**，動手前讀它 |
| `~/.kiro/powers/installed/<power>/POWER.md` | 總覽、工具清單、workflow 索引 | 需要確認工具精確名稱／參數時 |
| `~/.kiro/powers/repos/<power>/templates/` | JSON 範本（scaffold／preset／build config 等） | POWER.md 指示要載入範本時 |

讀取方式：`read` 工具支援 `~/.kiro/...` 這種寫法，直接用即可（已實測可行）。若失敗，有 `shell` 權限的 agent 可用 `ls "$HOME/.kiro/powers/installed"` 確認實際路徑。**不要在 prompt 或產出中寫死 `/Users/<某人>` 這種絕對路徑**。

> ⚠️ `templates/` 與 `hooks/` **只存在於 `repos/`**，`installed/` 只有 `POWER.md` + `steering/` + `mcp.json`。POWER.md 若叫你載入 `templates/...`，要去 `repos/` 找。

## Agent ↔ Power 對照表

Power repo 都在 GitHub `hoycdanny/<power 名稱>`（缺件時據此告知使用者要安裝哪一個）。

| Agent | 對應 Power | 領域 |
|-------|-----------|------|
| `unity-team` | `kiro-unity-accelerator` | Unity 場景／資產／Build／效能／架構／平台相容 |
| `godot-team` | `kiro-godot-accelerator` | Godot 場景架構／GDScript／Signal／TileMap／Export |
| `unreal-team` | `kiro-unreal-accelerator` | Unreal 關卡／Blueprint／材質／GAS／UE5 功能 |
| `cocos-team` | `kiro-cocos-accelerator` | Cocos 場景／節點元件／Prefab／跨平台 Build |
| `comfyui-team` | `kiro-comfyui-accelerator` | 影像生成：模型選型／prompt／sampler／ControlNet／放大／VRAM |
| `vfx-artist` | `kiro-comfyui-accelerator` | 同上（特效素材向；與 `comfyui-team` 共用同一個 Power） |
| `audio-team` | `kiro-ableton-accelerator` | 音樂製作：編曲／混音／樂理／鼓組律動／曲風 playbook |
| `krita-team` | `kiro-krita-accelerator` | 數位繪圖：畫布／筆刷／圖層／選取遮罩／構圖／匯出 |
| `slot-game-expert` | `kiro-slot-game-expert` | 老虎機：數學模型／RNG／認證／司法管轄區／負責任遊戲 |
| `fish-game-expert` | `kiro-fish-game-expert` | 魚機：命中判定 RNG／賠付／多人公平性／控分紅線／認證 |
| `wallet-systems-expert` | `kiro-gaming-wallet-expert` | 錢包後端：API／DB schema／幂等與鎖／對帳／可觀測性／合規 |

**沒有對應 Power 的 agent**（其餘 37 個，含 `producer`、5 個 Lead、其他設計／QA／發行角色）照原本的 prompt 運作，不需要也不應該去找 Power。

### 尚未可用的 Power

| Power | 狀態 |
|-------|------|
| `kiro-economy-balancing-expert` | repo 目前是空的（沒有 `steering/`）。`economy-designer` 與 `balance-tester` **暫時不要引用它**，維持自身 prompt 的知識；等它有內容再併入上表。 |

## 使用紀律

### 1. Steering-First：動手前先讀對應 steering

有對應 Power 的 agent，在執行任何多步驟操作前，**先讀該任務領域對應的 steering 檔**（每個 agent 的 prompt 裡有自己的「任務領域 → steering 檔案」對照表），再開始呼叫工具。

> ⚠️ **這條紀律必須靠 prompt 自律，沒有機制強制**。Unity／Cocos 等 Power 內含 `hooks/pre-*-tool.json`（preToolUse hook，作用是在呼叫工具前強迫先讀 steering），但依 `contracts.md` 的已知邊界，**subagent 執行環境不會觸發 Hooks**——本專案的 Pipeline 全走 subagent 委派，所以那道防護在這裡完全不生效。不要以為有 hook 在保護你。

### 2. 工具名稱與參數以 Power／實際錯誤訊息為準

Power 的 POWER.md 標明其工具清單是對真實連線驗證過的。若呼叫時仍收到 `Unknown action` 或參數驗證錯誤，**以錯誤訊息列出的合法值為最高權威**（MCP server 版本可能比 Power 文件更新），並回報使用者可能需要更新對應的 MCP 套件。不要憑印象猜第二次。

### 3. 缺 Power 時：誠實停下，不要降級硬做

讀不到對應 Power 的 steering 時：

1. 明確告知使用者「找不到 `<power 名稱>` 的 steering，路徑 `~/.kiro/powers/installed/<power>/steering/`」。
2. 告知安裝方式：Kiro → Powers 面板安裝，來源 `https://github.com/hoycdanny/<power 名稱>`。
3. **不要憑印象操作工具**，也不要假裝已完成任何 Editor／生成操作。純諮詢型問題（不碰工具）可以回答，但要標明「本次未取用 Power 知識，僅為一般性建議」。

這是 `contracts.md` 一貫的誠實聲明原則：能力邊界要講清楚，不要靜默降級。

### 4. Power 的知識邊界也要照實轉述

Power 內對「這個工具做不到什麼」的聲明（例如 unity Power 說明目前沒有雲端 build／裝置農場、unreal local MCP 工具集小於付費 Hosted 版），一律照實轉述給使用者，不要用推測填補。

## 變更紀錄

| 日期 | 變更 | 負責人 |
|------|------|--------|
| 2026-07-31 | 建立對照表，接入 11 個 Power；`kiro-economy-balancing-expert` 標記為尚未可用 | producer |
