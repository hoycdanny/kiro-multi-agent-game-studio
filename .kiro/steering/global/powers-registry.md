---
inclusion: always
---

# Kiro Powers 對照表（Powers Registry）

> **這份檔案是什麼**：本專案分成兩層——**Agent 是組織層**（誰做、何時做、用什麼 Contract 交付給誰），**Kiro Power 是領域知識層**（這個工具／這個領域實際上要怎麼正確做）。這份檔案是兩層之間的**唯一對照表**：告訴每個 agent「你的領域知識該去哪裡讀」。
>
> **為什麼要這樣分**：Power 的知識是對真實工具連線驗證過、且獨立於本專案持續更新的；agent prompt 裡手抄的工具細節會過時（本專案曾因此讓 `unity-team` 使用了數個不存在的 `manage_*` action）。所以**工具與領域知識的單一真相來源在 Power，不在 agent prompt**。
>
> **維護者**：`producer`（跨層對照的維護者）。新增 Power 或新增對應 agent 時，同步更新本表。

## 現況（2026-08-03）

**已安裝並可用：29 個 Power，全部有內容。** 48 個 agent 中有 **33 個**掛了對應 Power，其餘 15 個是協調與整合角色，不需要 Power。

**尚未可用的 Power 已清空——所有 Power 都可以引用了。**

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
| `~/.kiro/powers/installed/<power>/POWER.md` | 總覽、工具清單、Onboarding、steering 索引與各自的 trigger | 需要確認工具精確名稱／參數，或不確定該讀哪份 steering 時 |
| `~/.kiro/powers/repos/<power>/templates/` | JSON 範本（scaffold／preset／build config 等） | POWER.md 指示要載入範本時 |

讀取方式：`read` 工具支援 `~/.kiro/...` 這種寫法，直接用即可（已實測可行）。若失敗，有 `shell` 權限的 agent 可用 `ls "$HOME/.kiro/powers/installed"` 確認實際路徑。**不要在 prompt 或產出中寫死 `/Users/<某人>` 這種絕對路徑**。

> ⚠️ `templates/` 與 `hooks/` **只存在於 `repos/`**，`installed/` 只有 `POWER.md` + `steering/` + `mcp.json`。POWER.md 若叫你載入 `templates/...`，要去 `repos/` 找。

## Agent ↔ Power 對照表（可用）

Power repo 都在 GitHub `hoycdanny/<power 名稱>`（缺件時據此告知使用者要安裝哪一個）。

### 引擎與工具（Accelerator 型：有 MCP server）

| Agent | 對應 Power | 領域 |
|-------|-----------|------|
| `unity-team` | `kiro-unity-accelerator` | Unity 場景／資產／Build／效能／架構／平台相容 |
| `godot-team` | `kiro-godot-accelerator` | Godot 場景架構／GDScript／Signal／TileMap／Export |
| `unreal-team` | `kiro-unreal-accelerator` | Unreal 關卡／Blueprint／材質／GAS／UE5 功能 |
| `cocos-team` | `kiro-cocos-accelerator` | Cocos 場景／節點元件／Prefab／跨平台 Build |
| `blender-team` | `kiro-blender-accelerator` | Blender 建模／UV／材質／綁定／動畫／匯出（軸向與色彩空間） |
| `animator` | `kiro-blender-accelerator` | 同上（骨架與動畫向，讀 `rigging-and-skinning.md`／`animation-authoring.md`） |
| `technical-artist` | `kiro-blender-accelerator` | 同上（資產優化向，讀 `collider-and-lod.md`／`performance-and-limits.md`） |
| `comfyui-team` | `kiro-comfyui-accelerator` | 影像生成：模型選型／prompt／sampler／ControlNet／放大／VRAM |
| `vfx-artist` | `kiro-comfyui-accelerator` | 同上（特效素材向；與 `comfyui-team` 共用同一個 Power） |
| `krita-team` | `kiro-krita-accelerator` | 數位繪圖：畫布／筆刷／圖層／選取遮罩／構圖／匯出 |
| `audio-team` | `kiro-ableton-accelerator` | 音樂製作：編曲／混音／樂理／鼓組律動／曲風 playbook |
| `ui-ux-team` | `figma` | Figma 版面／Design Token 萃取／component 對應／切圖 handoff |

### 遊戲類型 Domain Expert（Knowledge Base 型：純知識）

| Agent | 對應 Power | 領域 |
|-------|-----------|------|
| `slot-game-expert` | `kiro-slot-game-expert` | 老虎機：數學模型／RNG／認證／司法管轄區／負責任遊戲 |
| `fish-game-expert` | `kiro-fish-game-expert` | 魚機：命中判定 RNG／賠付／多人公平性／控分紅線／認證 |
| `rpg-systems-expert` | `kiro-rpg-systems-expert` | RPG：傷害公式三類的數值行為／等級曲線／掉落長尾／技能樹 trap 判定 |
| `shooter-expert` | `kiro-shooter-expert` | 射擊：TTK 斷崖／命中判定架構／後座力模型／武器支配性檢定 |
| `card-game-expert` | `kiro-card-game-expert` | 卡牌：超幾何抽牌機率／費用強度曲線／power creep 量化偵測／HHI meta 多樣性 |
| `puzzle-match3-expert` | `kiro-puzzle-match3-expert` | 三消：可解性三層／board 生成拒絕率／連鎖期望／通關率敏感度 |
| `platformer-expert` | `kiro-platformer-expert` | 平台：跳躍物理反推／輸入寬容三機制／metroidvania gating 死鎖檢測 |
| `rhythm-expert` | `kiro-rhythm-expert` | 節奏：音訊時間軸權威／audio 與 input offset 分離／判定窗與抖動 |
| `strategy-expert` | `kiro-strategy-expert` | 策略：四子類型核心約束／相剋矩陣失衡檢測／塔防波次耦合／AI 難度公平性 |
| `simulation-expert` | `kiro-simulation-expert` | 模擬經營：生產鏈與供需收斂／資源閉環／系統交互 |
| `roguelike-expert` | `kiro-roguelike-expert` | Roguelike：程序生成／build synergy 上限／meta 進度平衡 |
| `narrative-adventure-expert` | `kiro-narrative-adventure-expert` | 敘事冒險：分支結構型態／旗標設計／可達性與死路驗證 |
| `mmo-expert` | `kiro-mmo-netcode-expert` | 多人：scope 分級（T1-T4）／伺服器權威／頻寬與容量模型／延遲補償取捨 |

### 跨領域專業（Knowledge Base 型）

| Agent | 對應 Power | 領域 |
|-------|-----------|------|
| `economy-designer` | `kiro-economy-balancing-expert` | 經濟：貨幣分層／sink-source 閉環／抽卡期望成本／進度曲線 |
| `balance-tester` | `kiro-economy-balancing-expert` | 同上（讀 `simulation-methodology.md`：模型建構／樣本量／收斂判斷／統計量） |
| `compliance-release` | `kiro-game-compliance-expert` | 合規：分級／隱私／平台送審／商店素材／揭露義務（含 casino 牌照流程） |
| `wallet-systems-expert` | `kiro-gaming-wallet-expert` | 錢包後端：API／DB schema／幂等與鎖／對帳／可觀測性／合規 |
| `systems-programmer` | `kiro-game-systems-expert` | 核心系統：存檔封套與遷移鏈／atomic write／資源所有權／事件風暴／狀態爆炸 |
| `localization-team` | `kiro-i18n-expert` | 多語：字串串接與複數規則／CJK 斷行禁則／RTL 鏡像／字型 subset 與豆腐塊 |
| `devops-team` | `kiro-game-devops-expert` | CI 出包：四引擎 headless build 結構／產物驗證八項／版本號／Git LFS 陷阱 |
| `usability-tester` | `kiro-usability-expert` | 可用性：評估框架／新手引導審查／卡關點分析／playtest 設計／五級證據等級 |

### 沒有對應 Power 的 agent（15 個）

`producer`、`creative-director`、5 個 Lead（`design-lead`／`domain-lead`／`art-lead`／`tech-lead`／`qa-lead`）、`game-designer`、`level-designer`、`narrative-designer`、`combat-designer`、`functional-tester`、`performance-tester`、`marketing-team`、`ui-programmer`。

這些照原本的 prompt 運作，不需要也不應該去找 Power。**Lead 與 Producer 的價值在跨 Specialist 的協調與取捨判斷，那是本專案的知識，不在任何 Power 裡。**

`functional-tester` 與 `performance-tester` 沒有 Power 是刻意的：功能測試的方法依專案而異、效能量測依引擎的 profiler 而異，兩者的知識分散在各引擎 Power 裡（例如 `kiro-unity-accelerator` 的效能章節），不該再複製一份。

## 使用紀律

### 1. Steering-First：動手前先讀對應 steering

有對應 Power 的 agent，在執行任何多步驟操作前，**先讀該任務領域對應的 steering 檔**，再開始呼叫工具或產出規格。

不確定該讀哪一份時：**先讀該 Power 的 `POWER.md`**，它的「Steering 索引」列出每份的 `trigger`（什麼情況該讀它）與 `description`。

> ⚠️ **這條紀律必須靠 prompt 自律，沒有機制強制**。Unity／Cocos 等 Power 內含 `hooks/pre-*-tool.json`（preToolUse hook，作用是在呼叫工具前強迫先讀 steering），但依 `contracts.md` 的已知邊界，**subagent 執行環境不會觸發 Hooks**——本專案的 Pipeline 全走 subagent 委派，所以那道防護在這裡完全不生效。不要以為有 hook 在保護你。

### 2. 工具名稱與參數以 Power／實際錯誤訊息為準

Power 的 POWER.md 標明其工具清單是對真實連線驗證過的。若呼叫時仍收到 `Unknown action` 或參數驗證錯誤，**以錯誤訊息列出的合法值為最高權威**（MCP server 版本可能比 Power 文件更新），並回報使用者可能需要更新對應的 MCP 套件。不要憑印象猜第二次。

### 3. 缺 Power 時：誠實停下，不要降級硬做

讀不到對應 Power 的 steering 時：

1. 明確告知使用者「找不到 `<power 名稱>` 的 steering，路徑 `~/.kiro/powers/installed/<power>/steering/`」。
2. 告知安裝方式：Kiro → Powers 面板安裝，來源 `https://github.com/hoycdanny/<power 名稱>`。
3. **不要憑印象操作工具**，也不要假裝已完成任何 Editor／生成操作。純諮詢型問題（不碰工具）可以回答，但要標明「本次未取用 Power 知識，僅為一般性建議」。

### 4. 信心等級要照實轉述

Knowledge Base 型 Power 普遍用三級標記：

| 等級 | 意義 | 轉述時 |
|------|------|-------|
| `HIGH` | 可用數學推導或有明文標準 | 可直接當結論用 |
| `MEDIUM` | 廣泛採用的設計慣例，非唯一解 | 要一併說出「什麼前提改了建議就會變」 |
| `UNVERIFIED` | 來自訓練資料的產業數字，未查證且隨時間變動 | **必須明說需要使用者用自家數據校準**，不要當事實講 |

Power 內對「這個工具做不到什麼」的聲明（例如 blender Power 說明未實測引擎端匯入行為、mmo Power 說明未實測任何引擎 API），一律照實轉述，不要用推測填補。

### 5. 兩個 Power 都相關時，各取所長不要重複

有些任務跨兩個 Power。既有 Power 之間已寫好分界，照它們的交叉引用走：

| 任務 | 主 Power | 借用 Power |
|------|---------|-----------|
| 射擊遊戲的連線命中判定 | `kiro-shooter-expert`（TTK／彈道） | `kiro-mmo-netcode-expert`（`latency-compensation.md`） |
| 任何類型的抽卡／轉蛋變現（卡牌／三消／RPG） | 對應類型 Power（遊戲平衡） | `kiro-economy-balancing-expert`（`gacha-and-lootbox.md`） |
| 任何類型的模擬驗證 | 對應類型 Power（模擬情境與判準） | `kiro-economy-balancing-expert`（`simulation-methodology.md` 通用方法論） |
| 局內資源產出與消耗的閉環（三消步數、模擬經營物資） | 對應類型 Power（局內規則） | `kiro-economy-balancing-expert`（`sink-source-modeling.md`） |
| 老虎機／魚機的送審 | 對應 casino Power（數學與 RNG） | `kiro-game-compliance-expert`（牌照與文件流程） |
| 節奏遊戲的音樂本身 | `kiro-rhythm-expert`（判定與譜面） | `kiro-ableton-accelerator`（編曲混音） |

## 變更紀錄

| 日期 | 變更 | 負責人 |
|------|------|--------|
| 2026-07-31 | 建立對照表，接入 11 個 Power；`kiro-economy-balancing-expert` 標記為尚未可用 | producer |
| 2026-07-31 | 18 個新 Power repo 建立骨架；全部標記為尚未可用 | producer |
| 2026-08-03 | **14 個 Power 完成內容並安裝，總計 25 個可用**；對照表依 Accelerator／Domain Expert／跨領域重新分組；新增信心等級轉述紀律與跨 Power 分工表；尚未可用縮減為 4 個（`game-systems`／`i18n`／`game-devops`／`usability`） | producer |
| 2026-08-03 | **最後 4 個 Power 完成並安裝，總計 29 個、全部可用**；「尚未可用」章節清空；33 個 agent 掛 Power，15 個協調角色刻意不掛 | producer |
| 2026-08-03 | 對磁碟核對 agent prompt 引用的 **376 個 steering 檔名，零虛構**；跨 Power 分工表補上實際存在但先前未列的兩個借用（RPG 的抽卡、局內 sink-source） | producer |
