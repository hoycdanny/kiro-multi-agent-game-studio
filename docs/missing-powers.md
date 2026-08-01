# 尚缺的 Kiro Power：建置規格

> 這是 [Kiro Multi-Agent Game Studio](../README.md) 的深入文件之一。完整索引見 README 的「深入文件（Reference）」。

本專案 48 個 agent 裡目前只有 11 個有 Power 支撐（對照表見 `.kiro/steering/global/powers-registry.md`）。其餘角色的知識還寫在自己的 prompt 裡——**這是本專案已證實會出問題的形態**：`unity-team` 曾累積 7 處實際不存在的 `manage_*` action，其中「連線自檢先讀 `project_info`」還是基於一個根本不該假設存在的 resource。

這份文件為每一個尚缺的 Power 提供**可直接開工的建置規格**：受益 agent、Power 型態、建議的 steering 檔案清單與各檔該寫什麼、驗證來源、以及完成後要在本專案改哪裡。

---

## 先讀：兩種 Power 型態

從既有 11 個 Power 觀察到的分野，新建時照同一套慣例走，維護才不會分裂：

| | **Guided MCP Power** | **Knowledge Base Power** |
|---|---|---|
| 例子 | `kiro-unity-accelerator`、`kiro-comfyui-accelerator`、`kiro-krita-accelerator` | `kiro-slot-game-expert`、`kiro-fish-game-expert`、`kiro-gaming-wallet-expert` |
| 有 `mcp.json` | 有（宣告要驅動的 MCP server） | 無（或 `mcpServers` 為空） |
| 入口 steering | `<domain>-general.md` 或 `mcp-workflow.md` | `advisory-engagement.md`（該問什麼前提） |
| 必備 steering | `verification-policy.md`（操作後怎麼確認真的生效）、`troubleshooting.md` | `verification-policy.md`（來源層級與時效標註） |
| `templates/` | JSON preset／scaffold／config | 規格範本、檢查清單 |
| `hooks/` | `preToolUse` 守衛，強制先讀 steering | 通常不需要 |

> ⚠️ **`hooks/` 在本專案不生效**：subagent 不觸發 Hooks（見 `powers-registry.md`）。Power 裡放 hook 對其他使用情境有用，但別指望它在這條 Pipeline 上保護你——steering-first 要寫進 POWER.md 的行為規範裡。

### 每個 Power 都該有的三份

1. **入口檔**（`*-general.md` 或 `advisory-engagement.md`）——基準原則 + 該問什麼 + 什麼情況該轉給別人
2. **`verification-policy.md`**——這是你既有 Power 最有價值的設計。Guided MCP 版寫「操作後怎麼確認真的生效」；Knowledge Base 版寫「來源層級（runtime > 官方文件 > model card > 社群）與會過期的斷言清單」
3. **`language-zh-tw.md`**（若沿用既有慣例）

---

## 完成一個 Power 後，要在本專案做的四件事

每一節下方的「接入步驟」都是這四件事的具體版本，通用流程如下：

1. **安裝** Power（Kiro → Powers 面板，來源 `https://github.com/hoycdanny/<power 名稱>`）
2. **`.kiro/steering/global/powers-registry.md`**：在「Agent ↔ Power 對照表」加一列
3. **對應 agent 的 `.md`**：加上「領域知識來源」章節與「任務領域 → steering 檔案」對照表（照 `unity-team.md` 的格式抄），並在規範衝突優先順序寫明「錯誤訊息 > Power > 本專案規範」
4. **刪掉該 agent prompt 裡被 Power 取代的手抄知識**——這一步最容易被跳過，但不刪就等於留了第二份會過時的副本，整個整合的意義就沒了

驗證方式：用 subagent 委派該 agent 做一次診斷任務，要它回報「讀了哪些 steering、引用其中一句具體內容」。若它答不出具體內容，就是沒真的讀到。

---

# P1：最高優先（3 個）

## `kiro-blender-accelerator`

| | |
|---|---|
| **受益 agent** | `blender-team`、`animator`、`technical-artist`（3 個，本專案最大的單一缺口） |
| **Power 型態** | Guided MCP |
| **MCP server** | `blender-mcp`（已在 `.kiro/settings/mcp.json`） |
| **參考範本** | `kiro-unity-accelerator`（同為 Editor 驅動型，工具面也類似） |

### 為什麼需要

三個 agent 共用 `blender-mcp`，但目前**零 Power 支撐**：UV 展開策略、骨架命名、蒙皮權重、匯出軸向與單位換算，全寫在 prompt 裡。而 3D 管線的錯誤特別隱性——軸向或 scale 錯了，模型進引擎才發現，且往前追不到是哪一步錯的。

這也是唯一「三個 agent 一次受益」的 Power，投報率最高。

### 建議 steering 檔案

| 檔名 | 內容 |
|------|------|
| `blender-general.md` | 入口。基準原則、MCP 連線健康檢查（用哪個輕量唯讀呼叫）、Blender 版本間的 `bpy` API 變動注意事項、**操作安全：不要覆蓋使用者既有場景** |
| `scene-inspection.md` | 動手前先看清狀態：現有物件、集合階層、修改器堆疊、既有 UV 與材質。對應既有 Power 的 `session-inspection` 慣例 |
| `modeling-workflow.md` | 建模流程、拓樸原則（四邊面優先、避免非流形與重疊頂點）、遊戲用網格的簡化取捨 |
| `uv-unwrapping.md` | 展開策略、接縫規劃、重疊檢查（鏡射是例外要標明）、UDIM、texel density 一致性 |
| `material-and-texture.md` | 材質節點、套用 PBR 通道；**色彩空間陷阱：albedo 用 sRGB、normal/roughness/metallic 必須 Non-Color**（這是最常見的靜默錯誤） |
| `rigging-and-skinning.md` | 骨架命名慣例、階層乾淨度、權重繪製、**Humanoid retarget 相容性要求**（影響能不能在引擎重用動畫） |
| `animation-authoring.md` | clip 切分、frame range、fps、是否 loop、root motion 與否、NLA 使用 |
| `collider-and-lod.md` | 簡化 collider mesh 的做法、凸包 vs 三角網格取捨、LOD 產生與面數階梯 |
| `export-settings.md` | `.fbx` / `.glb` 匯出參數；**軸向轉換（Blender Z-up ↔ Unity/Unreal Y-up 或 Z-up）與單位／scale 慣例**——這是進引擎後最常爆的一環，要給各引擎的對照表 |
| `python-scripting.md` | `bpy` 腳本模式、避免依賴 UI context 的寫法（headless 會失敗）、批次處理 |
| `performance-and-limits.md` | 大型網格與高成本修改器、操作逾時、記憶體限制 |
| `verification-policy.md` | 操作後怎麼確認真的生效：匯出檢視、讀回統計數字（面數／UV 島數／骨骼數）比對，**不要只信 API 回傳成功** |
| `troubleshooting.md` | 連線失敗、add-on 未啟用、匯出檔案打不開、進引擎後比例／方向錯誤的症狀對照 |
| `language-zh-tw.md` | 語言慣例（沿用既有 Power） |

### 驗證來源

Blender 官方 Manual 與 Python API 文件（`bpy` 版本差異）、各引擎官方的模型匯入文件（Unity Model Importer、Unreal FBX Import、Godot 匯入、Cocos 模型資源）——**軸向與 scale 的權威來源是引擎端文件，不是 Blender 端**。

### 接入步驟

1. 安裝 Power
2. `powers-registry.md` 加三列（`blender-team` / `animator` / `technical-artist` 都指向這個 Power）
3. 三個 agent 各加「領域知識來源」+ 任務領域對照表。分工建議：`blender-team` 讀 modeling／uv／material／collider-lod／export；`animator` 讀 rigging／animation／export；`technical-artist` 讀 material／performance／export／python-scripting
4. 刪掉三個 agent prompt 裡的手抄 3D 知識。**特別注意 `asset-standards.md` 的 poly budget 與命名規範要留在本專案**（那是專案規範不是工具知識）

---

## `kiro-economy-balancing-expert`

| | |
|---|---|
| **受益 agent** | `economy-designer`、`balance-tester` |
| **Power 型態** | Knowledge Base |
| **MCP server** | 無 |
| **參考範本** | `kiro-slot-game-expert`（同為數學／驗證導向） |

### 為什麼需要

repo（`hoycdanny/kiro-economy-balancing-expert`）**已經存在但是空的**——沒有 `POWER.md`、沒有 `steering/`。本專案的 `powers-registry.md` 已經把它標為「尚未可用」。補起來的邊際成本最低，因為 repo 骨架和 registry 條目都已就位。

`balance-tester` 特別需要它：目前它要跑 Monte Carlo 驗證 F2P 經濟，但「該建什麼模型、跑幾次、看哪個統計量」沒有任何知識來源。

### 建議 steering 檔案

| 檔名 | 內容 |
|------|------|
| `advisory-engagement.md` | 入口。該問什麼前提：F2P 還是買斷、目標 ARPPU／LTV、留存目標、有沒有 PvP（會改變經濟設計）、目標市場（影響定價與抽卡合規） |
| `economy-general.md` | 基準原則、貨幣分層（硬貨幣／軟貨幣／能量／材料）、為什麼要分層 |
| `currency-design.md` | 貨幣種類與職責切分、匯率設計、避免貨幣之間可自由套利 |
| `sink-source-modeling.md` | 產出（source）與消耗（sink）建模、通膨偵測、長期玩家的貨幣囤積問題 |
| `progression-curves.md` | 等級／升級成本曲線公式（線性／指數／分段），以及曲線形狀如何對應到目標遊玩時長 |
| `monetization-models.md` | IAP 商品階梯、首購獎勵、Battle Pass、訂閱制、廣告變現與遊戲性的衝突 |
| `gacha-and-lootbox.md` | 抽卡機率模型、pity／保底機制的數學、期望成本計算；**機率公示的合規要求**（與 `kiro-game-compliance-expert` 銜接，此處只寫數學，法規細節交叉引用） |
| `retention-and-funnel.md` | D1／D7／D30 留存定義、轉換漏斗、指標的正確算法與常見誤用 |
| `simulation-methodology.md` | **給 `balance-tester` 用的核心檔**：怎麼把經濟建成可模擬的模型、要跑幾次才收斂、該看哪些統計量（均值／分位數／變異）、隨機種子管理 |
| `balance-verification.md` | 驗證政策：什麼叫「平衡」、失衡的量化訊號、什麼情況下模擬結果不可信 |
| `pricing-psychology.md` | 錨定、誘餌商品、階梯定價的實務；同時標明**哪些做法在部分市場屬於暗黑模式**且有法規風險 |
| `troubleshooting.md` | 模擬不收斂、數值爆炸、模型與實際數據不符的排查 |
| `language-zh-tw.md` | 語言慣例 |

### 驗證來源

平台官方的變現政策（App Store Review Guidelines、Google Play 政策的付費與抽卡條款）、各地抽卡機率公示法規（日本、中國、韓國、比利時／荷蘭對 loot box 的規範差異）、公開的遊戲經濟設計文獻。**留存與 ARPPU 的業界基準會隨時間變動，要標註時效**。

### 接入步驟

1. 建立 `POWER.md` + steering，安裝
2. `powers-registry.md` 把它從「尚未可用的 Power」區塊**移到主對照表**，加兩列（`economy-designer`、`balance-tester`）
3. 兩個 agent 加「領域知識來源」+ 對照表。`balance-tester` 重點指向 `simulation-methodology.md` 與 `balance-verification.md`
4. 刪掉兩個 agent prompt 裡的手抄經濟知識

---

## `kiro-game-compliance-expert`

| | |
|---|---|
| **受益 agent** | `compliance-release` |
| **Power 型態** | Knowledge Base |
| **MCP server** | 無（該 agent 有 `web` 權限可查現行政策） |
| **參考範本** | `kiro-slot-game-expert` 的 `jurisdiction-matrix.md` / `certification-prep.md` 結構 |

### 為什麼需要

這個領域有**大量可驗證的官方來源**，而且錯誤代價高：分級問卷答錯會被退件、隱私揭露漏了會下架、兒童導向遊戲的付費限制違規會有罰則。性質和你已經寫得很好的 `slot-game-expert` 完全一樣——都是「法規密集 + 有官方文獻 + 會變動」。

目前 `compliance-release` 的知識全在 prompt 裡，且法規變動頻繁，正是最需要外部維護的類型。

### 建議 steering 檔案

| 檔名 | 內容 |
|------|------|
| `advisory-engagement.md` | 入口。該問什麼：目標市場與平台、目標年齡層、是否含 UGC／廣告／IAP／抽卡／社交功能、是否收集個資 |
| `compliance-general.md` | 基準原則、**「我不是法律顧問」的邊界聲明**、什麼情況必須請當地律師 |
| `age-rating-systems.md` | IARC 統一問卷與各地映射（ESRB／PEGI／CERO／USK／ACB／GRAC）的差異、問卷題目的判定邏輯、常見誤答 |
| `privacy-regulations.md` | GDPR／COPPA／CCPA／各地個資法在遊戲的實際適用點：同意機制、兒童年齡閘、資料最小化、跨境傳輸 |
| `platform-submission.md` | App Store／Google Play／Steam／主機平台的送審檢查清單與**常見退件原因** |
| `store-assets-spec.md` | 截圖尺寸與數量、預覽影片長度與規格、文案字數限制、各平台差異 |
| `data-collection-disclosure.md` | SDK 揭露義務、Apple Privacy Manifest／App Privacy、Google Data Safety Form 的填寫對應 |
| `monetization-disclosure.md` | loot box／抽卡機率公示要求（按市場）、訂閱條款揭露、兒童導向的付費限制 |
| `accessibility-requirements.md` | 各平台無障礙要求與建議實作 |
| `casino-licensing.md` | casino 牌照與認證流程協調（**只寫流程與文件，數學與 RNG 交叉引用 slot／fish Power，不重複**） |
| `ugc-and-moderation.md` | 含 UGC 時的審核義務、通報機制、平台要求 |
| `incident-and-takedown.md` | 下架、違規通知、資料外洩通報時限 |
| `verification-policy.md` | **這個 Power 最重要的一份**：來源層級（平台官方文件 > 監管機關公告 > 產業指引 > 社群）、每條斷言要標註查核日期、明確列出「會過期的斷言清單」 |
| `language-zh-tw.md` | 語言慣例 |

### 驗證來源

IARC、ESRB、PEGI、CERO、USK 官網；Apple App Review Guidelines 與 App Privacy 文件；Google Play 政策中心與 Data Safety；Steamworks 文件；GDPR／COPPA／CCPA 條文。**全部都要標查核日期**——這個領域三個月就可能變。

### 接入步驟

1. 建立 Power，安裝
2. `powers-registry.md` 加一列
3. `compliance-release.md` 加「領域知識來源」+ 對照表；保留它 `web` 權限的用途說明（查現行政策是刻意保留的能力，Power 提供框架、web 補最新變動）
4. 刪掉 prompt 裡的手抄法規清單

---

# P2：遊戲類型 Domain Expert（11 個）

**建議只補你實際會做的類型。** 一個沒有 Power 的 Domain Expert 價值接近零（內容不超過基礎模型已知範圍，卻佔 Agent Selector 位置與 context），但補一個就多一條能真正上線的類型線。

以下全部是 **Knowledge Base 型態、無 MCP server、參考範本 `kiro-slot-game-expert`**，入口檔一律 `advisory-engagement.md`，必備 `verification-policy.md` 與 `language-zh-tw.md`。接入步驟都相同：安裝 → `powers-registry.md` 加一列 → 對應 agent 加「領域知識來源」+ 對照表 → 刪掉 prompt 裡的手抄知識。

## `kiro-mmo-netcode-expert` → `mmo-expert`

**為什麼優先**：這是 11 個裡最容易寫錯、且錯了代價最大的。伺服器權威模型設計錯誤在後期幾乎不可能補救。

| 檔名 | 內容 |
|------|------|
| `advisory-engagement.md` | 該問：同時在線目標、PvP 還是 PvE、是否需要無縫世界、延遲容忍度、預算 |
| `mmo-general.md` | 基準原則、**「先界定 scope」的務實提醒**（多數專案不需要真正的 MMO 架構） |
| `server-authority.md` | 權威模型、客戶端只送輸入不送結果、防作弊的基本前提 |
| `state-replication.md` | 狀態同步策略、快照 vs 增量、序列化與頻寬預算 |
| `interest-management.md` | 興趣管理、視野裁剪、分區與 AOI |
| `latency-compensation.md` | 客戶端預測、伺服器回溯、lag compensation、與手感的取捨 |
| `persistence-and-sharding.md` | 持久化、分片／分區、跨區遷移 |
| `anti-cheat.md` | 常見作弊手法與對應的伺服器端驗證 |
| `scaling-and-ops.md` | 水平擴展、部署拓樸、監控指標 |
| `verification-policy.md` | 負載測試方法、什麼數據才算證明架構可行 |

**驗證來源**：各引擎官方網路文件（Unity Netcode／Unreal Replication／Godot MultiplayerAPI）、公開的 netcode 技術文獻。

## `kiro-rpg-systems-expert` → `rpg-systems-expert`

**為什麼優先**：純數學、可驗證、且 `balance-tester` 能直接接手模擬。

| 檔名 | 內容 |
|------|------|
| `advisory-engagement.md` | 該問：單機還是連線、目標遊玩時長、是否有裝備掉落、隊伍制還是單角色 |
| `rpg-general.md` | 基準原則、系統之間的相依（屬性→傷害→掉落→進度是一個閉環） |
| `stat-and-level-curves.md` | 屬性設計、等級曲線公式、經驗需求曲線與目標時長的換算 |
| `damage-formulas.md` | 傷害公式類型（減法／除法／乘算）與各自的數值行為、極端值問題 |
| `skill-trees.md` | 技能樹結構、解鎖節奏、避免必選節點 |
| `loot-and-rarity.md` | 掉落表機率、稀有度階梯、期望掉落時間計算 |
| `status-effects.md` | 狀態效果疊加規則、持續時間、免疫與抗性 |
| `class-and-party.md` | 職業／隊伍角色分工、避免單一最優解 |
| `quest-and-progression.md` | 任務結構、主線支線比重、進度 gating |
| `verification-policy.md` | 怎麼把系統交給 `balance-tester` 模擬、什麼數據算平衡 |

## `kiro-card-game-expert` → `card-game-expert`

| 檔名 | 內容 |
|------|------|
| `advisory-engagement.md` | 該問：PvP 還是 PvE、卡池規模、是否有抽卡變現、實體還是純數位 |
| `card-game-general.md` | 基準原則、卡牌遊戲的三層設計（單卡／牌組／meta） |
| `cost-to-power-curve.md` | 費用與強度的對應曲線、各費用段要有合理選擇 |
| `keywords-and-mechanics.md` | 關鍵字設計、可組合性、避免機制爆炸與規則歧義 |
| `archetype-and-synergy.md` | archetype 設計、synergy 強度上限、確保 meta 多樣 |
| `power-creep-control.md` | 強度基準線、新卡評估流程、輪替（rotation）機制 |
| `deck-rules.md` | 牌組規則（張數／複本上限／構築限制）如何影響 meta |
| `draft-and-selection.md` | draft／選牌規則的機率與公平性 |
| `verification-policy.md` | 大量對戰模擬的設計（勝率、先手優勢、combo 出現率），交 `balance-tester` |

## `kiro-puzzle-match3-expert` → `puzzle-match3-expert`

| 檔名 | 內容 |
|------|------|
| `advisory-engagement.md` | 該問：關卡數量目標、是否有體力／步數經濟、是否含變現 |
| `puzzle-general.md` | 基準原則、可解性是不可妥協的前提 |
| `board-generation.md` | board 生成演算法、初始無自動消除、**可解性保證**方法 |
| `match-and-cascade.md` | 消除判定、連鎖規則、連鎖期望值 |
| `level-objectives.md` | 關卡目標類型與難度貢獻 |
| `difficulty-curve.md` | 難度曲線設計、步數／moves 經濟與通關率的關係 |
| `special-pieces.md` | 特殊棋子／道具的效果與強度平衡 |
| `solvability-proof.md` | 可解性驗證方法（求解器、蒙地卡羅通關率估計） |
| `verification-policy.md` | 通關率目標、模擬多少局才可信 |

## `kiro-rhythm-expert` → `rhythm-expert`

**為什麼優先**：判定窗與 offset 校正是高度技術性的問題，做錯遊戲直接不能玩，而且有標準做法。

| 檔名 | 內容 |
|------|------|
| `advisory-engagement.md` | 該問：目標平台（延遲差異大）、輸入方式、譜面來源（自製／匯入） |
| `rhythm-general.md` | 基準原則、**音訊時間軸才是唯一真相**（不要用 frame 計時） |
| `timing-windows.md` | 判定窗設計（Perfect／Great／Good 的毫秒範圍）與難度的關係 |
| `audio-input-offset.md` | audio offset 與 input offset 的差別、**校正流程設計**（讓玩家自己校） |
| `beatmap-authoring.md` | 譜面設計原則、密度曲線、與音樂結構的對應 |
| `difficulty-tiers.md` | 難度分級的量化指標（NPS、模式複雜度） |
| `scoring-and-combo.md` | 分數／連段／評價系統設計 |
| `platform-latency.md` | 各平台的音訊延遲特性與應對 |
| `verification-policy.md` | 怎麼量測實際端到端延遲、判定窗是否合理的驗證方法 |

## `kiro-platformer-expert` → `platformer-expert`

| 檔名 | 內容 |
|------|------|
| `advisory-engagement.md` | 該問：2D 還是 3D、是否 metroidvania（影響 gating 設計）、關卡數量 |
| `platformer-general.md` | 基準原則、**手感無法靠規格驗證，必須人類試玩**（誠實邊界） |
| `jump-physics.md` | 重力、跳躍高度／滯空時間、可變跳躍高度、終端速度的具體參數區間 |
| `input-forgiveness.md` | coyote time、jump buffer、corner correction——這些是「手感好」的實際來源 |
| `movement-tuning.md` | 加速／減速曲線、空中控制、衝刺 |
| `level-rhythm.md` | 關卡節奏、挑戰密度、安全區間隔 |
| `ability-gating.md` | metroidvania 的能力解鎖 gating、可達性設計、避免死鎖 |
| `hazards-and-enemies.md` | 機關與敵人配置、傷害回饋 |
| `verification-policy.md` | 哪些能量化（通關時間、死亡點分佈）、哪些只能人類判斷 |

## `kiro-shooter-expert` → `shooter-expert`

| 檔名 | 內容 |
|------|------|
| `advisory-engagement.md` | 該問：FPS 還是 TPS、PvP 還是 PvE、是否連線（決定命中判定架構） |
| `shooter-general.md` | 基準原則、手感需人類驗證的誠實邊界 |
| `hit-detection.md` | hitscan vs projectile 的取捨、連線下的權威判定（與 `mmo-expert` 銜接） |
| `weapon-stats.md` | 傷害／射速／彈匣／換彈的數值結構與 TTK 換算 |
| `recoil-and-spread.md` | 後座力模式、擴散、可控性設計 |
| `weapon-balance.md` | 武器間平衡、避免單一最優、情境分化 |
| `enemy-and-bot-ai.md` | 敵人／Bot 行為設計、難度調節 |
| `gunfeel.md` | 射擊回饋的組成（動畫／音效／震動／後座）與各自貢獻 |
| `verification-policy.md` | TTK 表驗證、命中率統計、哪些必須人類試玩 |

## `kiro-strategy-expert` → `strategy-expert`

| 檔名 | 內容 |
|------|------|
| `advisory-engagement.md` | 該問：RTS／回合制／4X／塔防（四者設計差異極大）、單機還是對戰 |
| `strategy-general.md` | 基準原則、四個子類型的核心差異 |
| `unit-counters.md` | 兵種相剋矩陣設計、避免單一無敵解、剪刀石頭布之外的深度 |
| `resource-economy.md` | 資源與生產經濟、擴張與軍事的取捨 |
| `ai-opponent.md` | AI 對手行為層級、難度來源（資訊優勢 vs 數值加成的公平性） |
| `map-and-vision.md` | 地圖設計、視野／迷霧、地形優勢 |
| `tower-defense.md` | 塔防專屬：波次曲線、塔數值與成本曲線、路徑設計 |
| `turn-structure.md` | 回合制專屬：行動點、先後手平衡 |
| `verification-policy.md` | 對戰模擬與勝率驗證、AI 難度的量化 |

## `kiro-simulation-expert` → `simulation-expert`

| 檔名 | 內容 |
|------|------|
| `advisory-engagement.md` | 該問：模擬經營／生存製作／沙盒、是否有結束條件、目標遊玩時長 |
| `simulation-general.md` | 基準原則、**湧現是目標但不可控**，要靠系統交互而非腳本 |
| `production-chains.md` | 生產鏈設計、供需平衡、瓶頸的刻意安排 |
| `resource-loops.md` | 資源循環、是否閉環、長期是否會崩壞 |
| `building-and-automation.md` | 建造系統、自動化的解鎖節奏與「玩家還有事做嗎」的平衡 |
| `survival-needs.md` | 生存需求（飢餓／溫度／耐久）的數值與壓力曲線 |
| `emergence-and-interaction.md` | 系統交互設計、如何製造湧現 |
| `progression-pacing.md` | 進度與解鎖節奏、避免中期空轉 |
| `verification-policy.md` | 長時間模擬驗證經濟是否收斂、崩壞偵測 |

## `kiro-roguelike-expert` → `roguelike-expert`

| 檔名 | 內容 |
|------|------|
| `advisory-engagement.md` | 該問：roguelike 還是 roguelite（有無 meta 進度）、單局時長目標 |
| `roguelike-general.md` | 基準原則、隨機性要「有意義」而非單純不確定 |
| `procedural-generation.md` | 關卡／地城／掉落的程序生成、可玩性保證、種子管理 |
| `build-synergy.md` | run 內 build 構築、synergy 強度上限、避免必勝組合 |
| `risk-reward-events.md` | 隨機事件、風險報酬設計 |
| `meta-progression.md` | roguelite 的永久解鎖設計、避免「靠 meta 硬過」削弱技巧價值 |
| `difficulty-scaling.md` | 單局內與跨局的難度縮放 |
| `verification-policy.md` | 大量 run 模擬（勝率、build 出現率、平均時長），交 `balance-tester` |

## `kiro-narrative-adventure-expert` → `narrative-adventure-expert`

| 檔名 | 內容 |
|------|------|
| `advisory-engagement.md` | 該問：視覺小說／點擊冒險／敘事分支、預估文字量、語言數（影響 i18n 成本） |
| `narrative-general.md` | 基準原則、**與 `narrative-designer` 的分工**（系統結構 vs 內容） |
| `branching-structure.md` | 分支敘事結構型態（樹／圖／樞紐）與各自的維護成本 |
| `flags-and-state.md` | 旗標與狀態變數設計、命名慣例、避免狀態爆炸 |
| `dialogue-trees.md` | 對話樹結構、工具選擇（Yarn／Ink 等）的取捨 |
| `choice-consequence.md` | 選擇後果設計、延遲後果、避免假選擇 |
| `endings-and-convergence.md` | 結局分歧與收斂策略、內容量控制 |
| `pacing-and-puzzles.md` | 敘事節奏、解謎與敘事的交錯 |
| `localization-readiness.md` | 為多語預留的結構（與 `localization-team` 銜接：變數插入、語序、字數膨脹） |
| `verification-policy.md` | 分支可達性驗證、死路／孤島偵測、旗標一致性檢查 |

---

# P3：工程與其他（4 個）

## `kiro-game-systems-expert` → `systems-programmer`

| | |
|---|---|
| **Power 型態** | Knowledge Base（引擎無關的可攜模式知識） |
| **參考範本** | `kiro-gaming-wallet-expert`（同為後端／系統設計導向） |

| 檔名 | 內容 |
|------|------|
| `systems-general.md` | 入口。基準原則、**引擎無關的設計要怎麼寫才能被四個引擎 Team 落地** |
| `save-system.md` | 序列化格式選擇、**存檔版本遷移**（最容易被忽略卻最痛）、損壞復原、雲端同步衝突 |
| `resource-management.md` | 資源生命週期、載入策略（預載／串流／按需）、參照計數與釋放 |
| `event-system.md` | 事件系統模式（觀察者／訊息匯流排／ScriptableObject 事件）取捨、避免 event leak |
| `object-pooling.md` | 物件池設計、什麼該池化什麼不該 |
| `state-machines.md` | 狀態機模式、階層式狀態機、與動畫狀態機的關係 |
| `dependency-management.md` | 依賴注入 vs 服務定位、循環依賴的預防 |
| `verification-policy.md` | 存檔遷移的測試方法、資源洩漏的偵測 |

## `kiro-i18n-expert` → `localization-team`

| | |
|---|---|
| **Power 型態** | Knowledge Base |

| 檔名 | 內容 |
|------|------|
| `i18n-general.md` | 入口。基準原則、抽字串的時機（越早越便宜） |
| `string-extraction.md` | 抽字串策略、key 命名慣例、避免字串拼接 |
| `locale-file-formats.md` | 各格式取捨、複數規則、變數插入與語序 |
| `cjk-typography.md` | **中日韓斷行規則**、標點擠壓、行首行尾禁則、字距 |
| `rtl-support.md` | 阿拉伯／希伯來的鏡像佈局、雙向文字混排 |
| `font-and-subset.md` | 字型選擇、CJK subset 以控制包體、動態字型 fallback |
| `text-expansion.md` | 各語言字數膨脹率與 UI 預留空間（德文最長是常見陷阱） |
| `voice-and-subtitle.md` | 配音與字幕的同步、多語配音的資產組織 |
| `verification-policy.md` | 缺漏 key 偵測、截斷偵測、pseudo-localization 測試法 |

## `kiro-game-devops-expert` → `devops-team`

| | |
|---|---|
| **Power 型態** | Knowledge Base（也可做成 Guided MCP 若要驅動 CI 工具） |

| 檔名 | 內容 |
|------|------|
| `devops-general.md` | 入口。基準原則、遊戲專案 CI 與一般軟體 CI 的差異（產物大、需要 Editor、授權） |
| `headless-build.md` | 四個引擎的 headless build CLI（Unity `-batchmode`、Unreal UAT、Godot `--export`、Cocos CLI）與各自的坑 |
| `build-artifacts.md` | 產物命名、版本號策略、符號檔保存、產物驗證（可執行性／大小回歸） |
| `asset-pipeline-ci.md` | 大型二進位與 Git LFS、資產快取、匯入時間優化 |
| `platform-signing.md` | 各平台簽章與憑證管理（不要把憑證寫進 repo） |
| `test-automation.md` | 在 CI 跑引擎測試框架的實務限制 |
| `release-channels.md` | 內部測試／beta／正式的通道管理 |
| `verification-policy.md` | build 健康驗證清單、什麼情況該擋下 release |

## `kiro-usability-expert` → `usability-tester`

| | |
|---|---|
| **Power 型態** | Knowledge Base |

| 檔名 | 內容 |
|------|------|
| `usability-general.md` | 入口。基準原則、**最重要的誠實聲明：agent 不能真的玩遊戲**，本 Power 提供評估框架而非替代真人測試 |
| `onboarding-evaluation.md` | 新手引導評估框架、首次遊玩的認知負荷檢查清單 |
| `friction-analysis.md` | 卡關點分析方法、從可觀察訊號（重試次數／停留時間）推論的限制 |
| `ui-heuristics.md` | 遊戲 UI 的可用性啟發式（不同於一般軟體） |
| `accessibility-review.md` | 色盲／字級／字幕／操作替代方案的檢查 |
| `playtest-protocol.md` | 怎麼設計一次有效的真人 playtest、該問什麼、該避免的引導性問題 |
| `verification-policy.md` | 哪些結論可從資料得出、哪些必須真人驗證 |

---

## 附錄：新 Power 的完成度檢查清單

建好一個 Power 後，用這份清單自檢：

- [ ] `POWER.md` 有 frontmatter（`name` / `displayName` / `description` / `keywords`）
- [ ] 有入口 steering（`*-general.md` 或 `advisory-engagement.md`），且 `POWER.md` 明確指示先讀它
- [ ] 有 `verification-policy.md`，寫明來源層級或操作驗證方法
- [ ] 每條可能過期的斷言都標了查核日期
- [ ] 明確寫出**能力邊界**（這個 Power 做不到什麼），本專案的 agent 會照實轉述
- [ ] Guided MCP 型：工具清單是對真實連線驗證過的，且標明「以錯誤訊息的合法值為最高權威」
- [ ] 安裝後 `~/.kiro/powers/installed/<name>/` 有 `POWER.md` + `steering/`
- [ ] `.kiro/steering/global/powers-registry.md` 已加列
- [ ] 對應 agent 已加「領域知識來源」+「任務領域 → steering 檔案」對照表
- [ ] **對應 agent prompt 裡被取代的手抄知識已刪除**（最容易漏的一步）
- [ ] 用 subagent 委派該 agent 做診斷任務，它能引用 steering 裡的具體內容
