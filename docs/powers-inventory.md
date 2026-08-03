# Kiro Powers 完整清冊與覆蓋率分析

> **English summary**: Complete inventory of the 29 Kiro Powers used by this project, grouped into Accelerator (tool-backed, 12 agents), Domain Expert (game-genre knowledge, 13 agents), and Cross-domain (8 agents). Also documents why 15 agents intentionally have no Power, the three-tier confidence marking used inside Power content (`HIGH` / `MEDIUM` / `UNVERIFIED`), and a coverage gap analysis identifying four areas with no Power coverage. The authoritative agent-to-Power mapping lives in `.kiro/steering/global/powers-registry.md`.
>
> **這份檔案是什麼**：29 個 Power 的完整清冊、為什麼 15 個 agent 刻意不掛 Power、Power 內容的信心等級制度，以及覆蓋率缺口分析。README 只放摘要，細節在這裡。
>
> **權威對照表**在 `.kiro/steering/global/powers-registry.md`（`inclusion: always`，所有 Agent 自動載入）。本文件是給人讀的說明，那份是給 Agent 讀的規範。

---

## 兩層架構

Agent 是**組織層**（誰做、何時做、用什麼 Contract 交付給誰），[Kiro Power](https://kiro.dev/docs/powers/) 是**領域知識層**（這個工具／領域實際上怎麼正確做）。

**29 個 Power 全部已安裝且有內容，323 份 steering、約 4.9 MB。** 48 個 Agent 中有 33 個掛了對應 Power，其餘 15 個是協調與整合角色，刻意不掛。

---

## 引擎與工具型（Accelerator，12 個 Agent）

這類 Power 對應一個真實的 MCP server，知識是對實際連線驗證過的。

| Agent | Power | steering | 這個 Power 解決什麼 |
|-------|-------|:--------:|-------------------|
| `unity-team` | `kiro-unity-accelerator` | 15 | 場景／資產／Build／效能／架構／平台相容 |
| `godot-team` | `kiro-godot-accelerator` | 13 | 場景架構／GDScript／Signal／TileMap／Export |
| `unreal-team` | `kiro-unreal-accelerator` | 11 | 關卡／Blueprint／材質／GAS／UE5 功能 |
| `cocos-team` | `kiro-cocos-accelerator` | 14 | 場景／節點元件／Prefab／跨平台 Build |
| `blender-team` | `kiro-blender-accelerator` | 15 | 建模／UV／材質／匯出。**軸向與色彩空間是最常靜默出錯的環節** |
| `animator` | 同上 | — | 讀 `rigging-and-skinning.md`／`animation-authoring.md` |
| `technical-artist` | 同上 | — | 讀 `collider-and-lod.md`／`performance-and-limits.md` |
| `comfyui-team` | `kiro-comfyui-accelerator` | 11 | 模型選型／prompt／sampler／ControlNet／放大／VRAM |
| `vfx-artist` | 同上 | — | 特效素材向（與 `comfyui-team` 共用） |
| `krita-team` | `kiro-krita-accelerator` | 13 | 畫布／筆刷／圖層／選取遮罩／構圖／匯出 |
| `audio-team` | `kiro-ableton-accelerator` | 11 | 編曲／混音／樂理／鼓組律動／曲風 playbook |
| `ui-ux-team` | `figma`（Kiro 官方推薦） | 3 | 讀取版面／萃取 Design Token／Code Connect／design system 規則 |

> `figma` Power 假設的是「Figma → 網頁前端程式碼」，而本專案要的是「Figma → 遊戲引擎原生 UI」。讀取版面與萃取 Token 照它做，**產出階段改用本專案的 handoff 規格**，不要直接產 HTML/CSS。

---

## 遊戲類型 Domain Expert（Knowledge Base，13 個 Agent）

純知識、無 MCP server。這類 Power 的價值在於**把設計問題變成可計算的數學**，而不是給通用建議。

| Agent | Power | steering | 這個 Power 的技術核心 |
|-------|-------|:--------:|---------------------|
| `slot-game-expert` | `kiro-slot-game-expert` | 12 | 數學模型／RNG／認證／司法管轄區矩陣／負責任遊戲 |
| `fish-game-expert` | `kiro-fish-game-expert` | 16 | 命中判定 RNG／賠付／多人公平性／控分紅線／認證 |
| `rpg-systems-expert` | `kiro-rpg-systems-expert` | 11 | 傷害公式三類的極端值分析、掉落長尾（P90 = 2.3 倍期望）、技能樹 trap 判定 |
| `shooter-expert` | `kiro-shooter-expert` | 10 | **TTK 斷崖**（HP 100 下傷害 34 需 3 發、33 需 4 發，TTK 差 33%）、後座力模型、武器支配性檢定 |
| `card-game-expert` | `kiro-card-game-expert` | 10 | 超幾何抽牌機率表、power creep 量化偵測、HHI meta 多樣性、關鍵字交互 `C(n,2)` |
| `puzzle-match3-expert` | `kiro-puzzle-match3-expert` | 11 | 可解性三層（第三層數學上不可證）、board 生成拒絕率、通關率敏感度差 37 倍 |
| `platformer-expert` | `kiro-platformer-expert` | 10 | 跳躍物理反推（`g = 2h/t²`）、輸入寬容三機制、gating 死鎖圖檢測 |
| `rhythm-expert` | `kiro-rhythm-expert` | 10 | 音訊時間軸權威（frame 計時 3 分鐘累積 1 秒）、audio 與 input offset 必須分離 |
| `strategy-expert` | `kiro-strategy-expert` | 10 | 四子類型核心約束、相剋矩陣失衡檢定、塔防波次與收入耦合、AI 難度公平性 |
| `simulation-expert` | `kiro-simulation-expert` | 10 | 生產鏈與供需收斂、資源閉環、長期崩壞偵測 |
| `roguelike-expert` | `kiro-roguelike-expert` | 9 | 程序生成正確性、種子架構、build synergy 上限、meta 進度平衡 |
| `narrative-adventure-expert` | `kiro-narrative-adventure-expert` | 14 | 分支結構型態與維護成本、旗標設計、可達性與死路驗證 |
| `mmo-expert` | `kiro-mmo-netcode-expert` | 11 | **scope 分級 T1–T4**（多數說要做 MMO 的專案其實需要 T2）、頻寬容量模型、延遲補償取捨 |

---

## 跨領域專業（Knowledge Base，8 個 Agent）

| Agent | Power | steering | 技術核心 |
|-------|-------|:--------:|---------|
| `economy-designer` | `kiro-economy-balancing-expert` | 13 | 貨幣分層／sink-source 閉環／抽卡期望成本與 pity 數學／進度曲線 |
| `balance-tester` | 同上 | — | 讀 `simulation-methodology.md`：樣本量反推 `n ≥ (1.96σ/ε)²`、收斂判斷、RNG stream 分流 |
| `compliance-release` | `kiro-game-compliance-expert` | 14 | 分級／隱私／送審／商店素材／揭露義務。**含 45 類「會過期的斷言清單」** |
| `wallet-systems-expert` | `kiro-gaming-wallet-expert` | 10 | API／DB schema／幂等與鎖／對帳／可觀測性／金流合規 |
| `systems-programmer` | `kiro-game-systems-expert` | 9 | 存檔封套與遷移鏈（逐版 `N-1` vs 捷徑 `N(N-1)/2`）／atomic write 步驟順序／事件風暴 `f^d` |
| `localization-team` | `kiro-i18n-expert` | 10 | 字串串接為何無解／CJK 禁則／RTL 鏡像／字型 subset 與豆腐塊 |
| `devops-team` | `kiro-game-devops-expert` | 9 | 四引擎 headless build／**產物驗證八項**（exit code 0 有七種失敗形態）／版本號／Git LFS |
| `usability-tester` | `kiro-usability-expert` | 8 | 五級證據等級／新手引導審查／卡關點分析／playtest 設計 |

---

## 為什麼有 15 個 Agent 刻意不掛 Power

這是設計決策，不是缺漏：

| Agent | 為什麼不需要 |
|-------|------------|
| `producer`、`creative-director` | 調度與願景是本專案的組織知識，不屬於任何領域 |
| 5 個 Lead（`design`／`domain`／`art`／`tech`／`qa`） | **價值來自跨 Specialist 的取捨判斷**。給 Lead 掛 Power 會讓它偏向那個領域，而選型時的中立性正是它存在的理由——你不可能問 `unity-team`「該不該用 Unity」 |
| `game-designer` | GDD 整合角色，領域知識分散在 13 個 Domain Expert Power 裡 |
| `level-designer` | 關卡設計知識已分佈在 platformer／strategy／puzzle／roguelike 各自的 Power |
| `ui-programmer` | UI 綁定的做法由各引擎 Power 覆蓋 |
| `functional-tester` | 功能測試方法依專案而異；CI 執行面在 devops Power |
| `performance-tester` | 效能量測依各引擎 profiler 而異，知識在各引擎 Power 的效能章節 |
| `narrative-designer` | 敘事**系統結構**在 narrative-adventure Power；本角色產出的是**內容** |
| `combat-designer` | 戰鬥數值在 shooter／rpg Power；本角色服務的是沒有專屬 Power 的類型 |
| `marketing-team` | 純文字產出，無工具依賴 |

---

## Power 內容的信心等級（引用前必讀）

Knowledge Base 型 Power 普遍用三級標記，Agent 應照實轉述：

| 等級 | 意義 | 佔比感受 |
|------|------|---------|
| `HIGH` | 可用數學推導或有明文標準（公式、組合數學、Unicode／CLDR 規則、POSIX 語意） | 數學部分幾乎全是 |
| `MEDIUM` | 廣泛採用的設計慣例，非唯一解。轉述時要一併說「什麼前提改了建議會變」 | 參數選擇多屬此類 |
| `UNVERIFIED` | 來自訓練資料的產業數字，未查證且隨時間變動 | **佔比不小**，見下 |

**`UNVERIFIED` 集中在四類**，引用時必須明說需要使用者用自家數據校準：

- 所有「業界平均」（留存率、ARPPU、常見 TTK 區間、coyote time 毫秒數、受測人數建議）
- 所有法規細節（分級問卷、平台政策、機率公示義務——`kiro-game-compliance-expert` 的 `UNVERIFIED` 是刻意佔多數的）
- 所有引擎端行為（各 Power 沒有連線可驗證引擎的匯入設定與 API）
- 所有平台延遲與硬體規格數字

---

## 架構聲明：知識庫在 repo 外，本 repo 只存路由

這一點是刻意的設計，不是實作偷懶：

| | 存什麼 | 在哪 |
|---|---|---|
| **本 repo** | **路由與組織**：哪個 agent 對應哪個 Power、該讀哪個 steering 檔名、什麼時機讀、缺件時怎麼回報 | `.kiro/` |
| **Kiro Power** | **知識本身**：那個 steering 檔案實際寫了什麼 | `~/.kiro/powers/installed/`（全機安裝，repo 外） |

可驗證的事實：323 份 Power steering 全部在 repo 外；repo 內對 Power 知識內容的字串搜尋零命中（測過 `Redlock`、`euler_ancestral`、`GPU Resident Drawer`、`krita_select_by_alpha` 等各 Power 獨有字串）；repo 內提到 Power 的檔案內容都是**引用路徑與檔名**，而非複製內容。

**為什麼不把知識放進來**：Power 的知識對真實工具連線驗證過，且獨立於本專案持續更新。複製進 repo 就會產生第二份會過時的副本——本專案已經因此吃過一次教訓：整合前 `unity-team.md` 有 7 處已失效的 API（`manage_asset(list)`、`manage_editor(action:"build")`、`manage_graphics(get_rendering_stats)` 等，Power 已標明這些 action 不存在），其中「連線自檢先讀 `project_info`」這一步本身就基於一個 Power 明確說「不要假設存在」的 resource。

**代價（誠實聲明）**：這讓本 repo **不是自足的**。clone 下來，33 個 agent 的知識層是空的，需要另外從 Powers 面板安裝 29 個 Power。目前沒有可機器檢查的 manifest 或 setup 腳本，只有文件說明與 `powers-registry.md` 的對照表。

### 三個必須知道的邊界

1. **Power 是全機安裝、不隨 repo 走**（在 `~/.kiro/powers/`）。clone 這個 repo 不會帶來知識層。
2. **缺 Power 時 Agent 會誠實停下**並回報安裝來源，不會憑印象操作工具、也不會靜默降級。
3. **Power 內含的 `hooks/`（preToolUse，強迫先讀 steering）在本專案不生效**——依官方文件 subagent 不觸發 Hooks，而本專案 Pipeline 全走 subagent 委派。**Steering-First 紀律靠 prompt 自律，沒有機制強制**，這正是 `unity-team` 當初累積 7 處失效 API 的同一個成因。

---

## 覆蓋率缺口分析（2026-08-03 實測）

**已驗證的事實**：29 個 Power 全部有 agent 引用（零孤兒）；33/48 個 agent 掛 Power；所有 steering 檔名對磁碟核對過（無虛構檔名）。

以下是**目前沒有 Power 覆蓋、且值得考慮補的四個缺口**。這不是待辦清單，是誠實的覆蓋率盤點——每一項都說明現在誰在頂替、以及不補的代價：

| 缺口 | 受影響 agent | 現在誰頂替 | 不補的代價 |
|------|------------|-----------|-----------|
| **跨引擎 profiling 方法論** | `performance-tester` | 各引擎 Power 的效能章節（分散、只有該引擎視角） | 效能數字有變異性，沒有方法論容易「優化了錯的東西」而看不出來。缺的是：該量什麼、frame budget 歸因、量測的統計有效性、平台專屬陷阱 |
| **格鬥／動作遊戲的近戰戰鬥** | `combat-designer` | 自身 prompt。shooter Power 只覆蓋射擊、rpg Power 只覆蓋數值 | frame data、hitbox/hurtbox、input buffer 與 cancel 窗、連段設計、hitstop 這些**沒有任何 Power 覆蓋**。13 類 Domain Expert 裡沒有格鬥類 |
| **敘事內容撰寫與工具** | `narrative-designer` | 自身 prompt。narrative-adventure Power 覆蓋的是**系統結構**不是內容 | Ink／Yarn／Twine 的語法與慣例、World Bible 結構、對話撰寫工藝，目前只能靠基礎模型知識 |
| **商店轉換與預告片結構** | `marketing-team` | 自身 prompt | 商店頁轉換要素、預告片 shot list 結構、press kit 組成，屬可累積的工藝知識 |

**同時盤點出的類型覆蓋缺口**：13 類 Domain Expert 沒有涵蓋 **格鬥、賽車、體育、恐怖、派對遊戲**。其中格鬥的機制最獨特（frame data 是一整套獨立學問），其餘四類目前由既有 expert 部分服務。要不要補取決於你實際會做什麼類型——**不建議為了覆蓋率而補**，48 個 agent 已經是需要謹慎管理的規模。

### 判斷是否值得補一個 Power 的三條標準

本專案的實測經驗：

1. **內容會不會超過基礎模型已知範圍？** 若一份 Power 寫出來的東西大語言模型本來就知道，它的價值接近零——只是把同樣的知識搬到另一個檔案。有價值的是**具體數字與推導**（例如 TTK 斷崖的臨界點清單）、**可驗證的 API 事實**（例如 Blender 5.x 已移除 `action.fcurves`）、**當前法規與日期**。
2. **錯誤的代價高不高？** 存檔遷移做錯會丟玩家進度、合規做錯會下架——這類優先。
3. **知識會不會過時？** 會過時的（工具 API、法規）更該放 Power，因為 Power 能獨立更新；不會過時的（數學）放哪都行。
