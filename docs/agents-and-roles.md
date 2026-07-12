# Agent 與角色

> 這是 [Kiro Multi-Agent Game Studio](../README.md) 的深入文件之一。完整索引見 README 的「深入文件（Reference）」。

## Slot Game Expert 詳解


> `design/slot-game-expert` 不是一個操作 MCP 工具的執行 Team，而是一個**純知識型 Domain Expert**，產出數學模型/RNG/認證合規規格，交給對應的引擎 Team 實作。

### 為什麼獨立於 `game-designer` 之外

老虎機開發涉及高度專業且風險敏感的知識（CSPRNG 選型、GLI 認證、負責任遊戲法規），這些不是一般遊戲設計師的日常知識範疇，錯誤的建議可能導致認證失敗甚至法規違規。因此獨立成一個專屬 Domain Expert，`game-designer` 遇到老虎機需求時會主動轉介，不會自己硬答。

### 涵蓋領域

- **數學模型設計**：Paytable、Virtual Reel 權重、RTP 計算、Volatility 調校、Hit Frequency、Bonus/Free Spin 的 RTP 貢獻
- **RNG 與遊戲邏輯**：CSPRNG 選型（依引擎不同）、種子管理、六階段 Spin Lifecycle、審計日誌欄位設計
- **認證合規**：GLI-11（實體機台）/ GLI-19（線上）標準、認證文件清單、市場法規、時程費用估算
- **負責任遊戲**：存款限制、自我排除（串接 GamStop/Spelpaus 等官方系統）、會話時間提醒、Autoplay 限制

### 引擎對應的 CSPRNG 選型（供快速查閱）

| 引擎 | CSPRNG API | 對應 Team |
|------|-----------|-----------|
| Unity | `System.Security.Cryptography.RandomNumberGenerator` | `engineering/unity-team` |
| Godot | `Crypto.generate_random_bytes()` | `engineering/godot-team` |
| Unreal Engine | OpenSSL `RAND_bytes()` | `engineering/unreal-team` |
| Cocos Creator | `crypto.getRandomValues()`（瀏覽器）/ `crypto.randomBytes()`（Node.js） | `engineering/cocos-team` |

> **核心規則**：CSPRNG 是唯一可接受的 RNG 類型，一般的 `Random()` / `Math.random()` / `FMath::RandRange` 都不具密碼學安全性，絕對不能用在正式上線的核心邏輯。

### 參考資料（完整清單見 `slot-game-expert.md` 內文，24 條已驗證官方文獻）

- [GLI Standards](https://gaminglabs.com/gli-standards/)
- [NIST SP 800-90A Rev.1](https://csrc.nist.gov/pubs/sp/800/90/a/r1/final)

---

## 遊戲類型 Domain Expert 一覽


本專案把「不同遊戲類型的設計專業」拆成 13 個 Domain Expert（都在 `design/`，`claude-sonnet-5`，純知識型、不操作引擎 MCP）。Producer 依需求關鍵字路由到對應專家，產出**系統規格與數值**交 `game-designer` 整合進 GDD、`balance-tester` 驗數值、引擎 Team 實作。

**共通模式**：每個專家被喚醒時都會先問清楚關鍵前提（單/多人、子類型、規模、平台），再產規格；數值一律標「初版，待模擬/實測調整」；完整規格見各自的 `.kiro/agents/design/*.md`。下面是「何時找它、交付什麼、最關鍵的專業點、跟誰協作」。

> 類型會**疊加**：一款遊戲常同時觸發多個專家（見末尾組合範例），由 Producer 串接。

#### 🎰 `slot-game-expert`（老虎機 / casino）
- **觸發**：老虎機 / slot / 拉霸 / casino
- **交付**：捲軸數學模型、Paytable、RTP/波動度、CSPRNG 指引、認證合規清單、負責任遊戲設計
- **關鍵點**：RTP 必須數學可證且模擬可重現；RNG 只接受 CSPRNG；牽涉牌照/GLI 認證
- **協作**：`balance-tester`（跑千萬次 spin 驗 RTP）、`compliance-release`（送審/牌照）、`comfyui-team`（符號美術，通常 2D 可跳過 Blender）
- **深度專章**：見上方「Slot Game Expert 詳解」（附 24 條官方文獻）

#### 🐟 `fish-game-expert`（魚機 / 捕魚）
- **觸發**：魚機 / 捕魚 / fish hunter
- **交付**：各魚種命中機率、賠付經濟、RTP、伺服器權威判定 RNG、合規
- **關鍵點**：命中/賠付必須**伺服器判定**（客戶端只做表演），否則可被作弊刷分；casino 市場需合規
- **協作**：`balance-tester`（驗 RTP/賠付經濟）、`mmo-expert`（伺服器權威）、`compliance-release`

#### 🔫 `shooter-expert`（射擊 FPS/TPS）
- **觸發**：射擊 / FPS / TPS / shooter
- **交付**：武器數值表（傷害/射速/彈匣/後座/擴散/衰減）、TTK 平衡、命中判定模型、敵人 AI、gunfeel 規格
- **關鍵點**：hitscan vs projectile 的選擇 + 爆頭倍率；多人時命中權威在伺服器（需 lag compensation）
- **協作**：`mmo-expert`（多人命中權威）、`balance-tester`（武器平衡）、`audio-team`（打擊音效節奏）

#### 🌐 `mmo-expert`（多人 / MMORPG）
- **觸發**：多人 / 連線 / MMO / MMORPG / co-op / PvP
- **交付**：netcode 架構、伺服器權威模型、狀態同步、持久化、防作弊、務實 scope 界定
- **關鍵點**：⚠️ 全功能 MMO 對 solo dev 極重，務必先砍成小規模 co-op/競技；伺服器權威是防作弊底線
- **協作**：幾乎所有帶多人的類型都會拉它（射擊/RPG/策略/魚機）+ 引擎 Team 做 netcode

#### ⚔️ `rpg-systems-expert`（RPG / ARPG）
- **觸發**：RPG / ARPG / 角色扮演
- **交付**：屬性系統、等級/經驗曲線、傷害公式、技能樹/職業、裝備與掉落表、任務/進度結構
- **關鍵點**：傷害公式與等級曲線要**可被模擬重現**；避免單一 build 碾壓、後期數值通膨
- **協作**：`balance-tester`（驗成長曲線）、`mmo-expert`（若線上）、掉落綁付費時 `economy-designer`+`compliance-release`

#### 🃏 `card-game-expert`（卡牌 / Deckbuilder）
- **觸發**：卡牌 / deckbuilder / TCG / autobattler
- **交付**：卡牌數值、資源曲線（費用/節奏）、archetype 與 combo、平衡準則、擴充包規劃
- **關鍵點**：combo 空間要豐富但無必勝解；資源曲線決定節奏；新卡牌對既有牌池的平衡衝擊
- **協作**：`balance-tester`（模擬對戰勝率）、`economy-designer`（開包/集換變現）、`compliance-release`（開包機率公示）

#### 🍬 `puzzle-match3-expert`（三消 / 解謎）
- **觸發**：三消 / 消除 / match-3 / 解謎 / merge
- **交付**：board 生成與**可解性保證**、消除/連鎖/特殊方塊規則、關卡難度曲線、步數/體力經濟
- **關鍵點**：每次生成/洗牌必須保證至少一步可消（無死局）；難關與喘息關交錯避免流失
- **協作**：`balance-tester`（模擬通關率/難度）、`economy-designer`（體力/加步道具變現）

#### 🏃 `platformer-expert`（平台 / metroidvania）
- **觸發**：平台 / 跳台 / metroidvania / 類銀河戰士
- **交付**：跳躍手感數值（重力/跳躍高度/可變跳/**coyote time**/**jump buffer**）、移動物理、關卡節奏、能力 gating
- **關鍵點**：平台遊戲成敗在手感——容錯幀（coyote/jump buffer）是「跳起來爽不爽」的關鍵，數值需引擎 team 實機微調
- **協作**：引擎 Team（實機調控制器）、`shooter-expert`（若有射擊型 boss）

#### 🎲 `roguelike-expert`（roguelike / roguelite）
- **觸發**：roguelike / roguelite / 肉鴿 / 程序生成
- **交付**：程序生成規則（種子可重現）、run 內 build/synergy 平衡、風險報酬事件、難度縮放、meta 永久進度
- **關鍵點**：保證每局可通關且不無聊（無必死開局、無空轉房間）；build 要有強 combo 但無必勝解
- **協作**：`balance-tester`（驗 build 強度分布/勝率）、視核心戰鬥併 `rpg`/`shooter`/`card` expert

#### 🏰 `strategy-expert`（RTS / 回合 / 4X / 塔防）
- **觸發**：策略 / RTS / 即時戰略 / 回合策略 / 4X / 塔防 / tower defense
- **交付**：單位/兵種數值與相剋、資源與生產經濟、AI 對手行為、（塔防）波次曲線與塔數值/性價比
- **關鍵點**：兵種相剋 + 成本/人口/生產時間構成平衡三角，避免主宰單位；經濟節奏決定爆兵時間點
- **協作**：`balance-tester`（模擬平衡）、`mmo-expert`（多人同步）

#### 🏭 `simulation-expert`（模擬經營 / 生存 / 沙盒）
- **觸發**：模擬經營 / tycoon / 生存 / 製作 crafting / 沙盒 / 自動化
- **交付**：生產鏈/合成樹、供需與價格模型、生存需求平衡、自動化/效率曲線、系統交互
- **關鍵點**：遊戲內經濟必須**收斂**（玩久了不通膨、不枯竭）；生產循環要交 `balance-tester` 跑長時程模擬
- **協作**：`balance-tester`（驗經濟收斂）、`economy-designer`（若含付費，遊戲內經濟 ≠ 變現經濟）、`mmo-expert`（多人共存）

#### 🎵 `rhythm-expert`（音樂節奏）
- **觸發**：音樂 / 節奏 / 音 game / rhythm
- **交付**：譜面設計、判定窗（ms）、**延遲校正流程**（audio/input offset）、計分/連段/評價
- **關鍵點**：音 game 成敗在同步——必須設計 offset 校正讓不同裝置/藍牙延遲都能對拍；判定對齊音訊時間軸而非畫面幀
- **協作**：**強綁 `audio-team`**（先拿曲子 BPM/拍點/段落才能鋪譜）、引擎 Team（實機驗同步）

#### 📖 `narrative-adventure-expert`（敘事 / 視覺小說 / 冒險）
- **觸發**：視覺小說 / VN / 敘事 / 冒險 / 點擊冒險 / 互動敘事
- **交付**：分支敘事結構（線性/樹狀/foldback）、旗標/狀態變數、對話樹、選擇後果與結局分歧、pacing
- **關鍵點**：用 foldback 控制分支爆炸讓選擇有意義又不失控；旗標命名/作用域要清楚方便引擎實作
- **協作**：**強綁 `localization-team`**（大量文字要及早標可翻譯字串）、`game-designer`（世界觀）、`audio-team`（配音）

### 組合範例（Producer 串接多個專家）
- **多人射擊 RPG**：`mmo-expert`（netcode）＋ `shooter-expert`（槍械手感）＋ `rpg-systems-expert`（養成）
- **付費開包卡牌**：`card-game-expert`（平衡）＋ `economy-designer`（變現）＋ `compliance-release`（機率公示）
- **含 roguellike 元素的動作遊戲**：`roguelike-expert`（生成/build）＋ `platformer-expert` 或 `shooter-expert`（核心戰鬥）
- **音樂節奏 + 敘事**（如節奏冒險）：`rhythm-expert`（判定）＋ `narrative-adventure-expert`（劇情）＋ `audio-team`
- **模擬經營手遊**：`simulation-expert`（生產循環）＋ `economy-designer`（IAP）＋ `localization-team`（多語）

> 沒有對應專家的類型（競速、格鬥、體育、派對、walking sim、idle/clicker…）走通用 `game-designer`；格鬥的 rollback netcode 併 `mmo-expert`。

---

## 團隊角色與職責


### Layer 0：Strategic（戰略層）

| Agent | 檔案 | 職責 |
|-------|------|------|
| Creative Director | `orchestration/creative-director.md` | 守護遊戲願景、創意方向最終仲裁、美術風格決定權 |

### Layer 1：Orchestration（指揮層）

| Agent | 檔案 | 工具 | 職責 |
|-------|------|------|------|
| Producer | `orchestration/producer.md` | read, write, shell（本機 git）, `@github`（Projects/issues，需連上） | 拆解任務、產出 Contract、指引分派、追蹤進度 |

> Creative Director 管「做什麼」，Producer 管「怎麼做」。

### Layer 2：Lead（品質守門 / review gate）

> 這一層是各領域的 review gate 與真相文件維護者；Producer 可在進入下一階段前委派它們審查。

| Lead | 檔案 | 職責 |
|------|------|------|
| Design Lead | `design/design-lead.md` | 核心設計（7 個常駐職能）整合 GDD、消矛盾、design-review gate |
| Domain Lead | `design/domain-lead.md` | 13 類遊戲類型 Domain Expert 的專業正確性審查與轉發（按需啟用，不進 GDD 整合，交 Design Lead） |
| Art Lead | `art/art-lead.md` | 維護 `style-guide.md`、跨美術 Team 一致性 review |
| Tech Lead | `engineering/tech-lead.md` | 技術架構決策、效能預算、跨引擎 code-review gate |
| QA Lead | `qa/qa-lead.md` | 測試策略、協調 functional/balance/performance tester、go/no-go |

> Audio Lead **刻意不獨立建立**：目前只有單一 `audio-team`，無需再加一層管理；音訊一致性已明確併入 `art-lead` 的職責（見 `art-lead.md` frontmatter 說明）。
>
> **Design Lead 與 Domain Lead 的分工**：Design Lead 管幾乎每個專案都會用到的 7 個核心設計職能；Domain Lead 管 13 個按遊戲類型按需啟用的專家（一個專案通常只用到其中 1-3 個）。Domain Lead 審類型的專業正確性（RTP 數學、netcode 架構等），Design Lead 審整體設計連貫性並最終整合進 GDD。詳見各自 agent 檔案的「與 XX 的分工」章節。

### Layer 3：Specialist Agents

#### Design Team（核心設計，7 個）

| Agent | 工具 | 產出 |
|-------|------|------|
| game-designer | read, write | GDD、系統規格、數值平衡表、Asset Spec（無專屬 expert 的類型走這條） |
| economy-designer | read, write | 經濟模型、F2P 數值、商城定價、IAP、Battle Pass |
| combat-designer | read, write | 通用戰鬥系統/技能設計/敵人 AI；FPS 交 shooter-expert、RPG 交 rpg-systems-expert，避免重複覆蓋 |
| level-designer | read, write | 關卡佈局、觸發器/事件設計、難度曲線；產出交對應引擎 Team 實際搭建（引擎無關，不綁定單一引擎 MCP） |
| narrative-designer | read, write | 世界觀、角色背景、主線/支線劇情、對話內容、World Bible（與 narrative-adventure-expert 分工：內容 vs 系統結構） |
| ui-ux-team | @figma, read, write | Wireframe、操作流程、新手引導、UI Layout、Design Token、互動狀態規格、切圖規格（合併原願景 ux-designer + ui-artist），見「Figma MCP 整合詳解」 |
| localization-team | read, write, shell | 多語系字串抽取、locale 檔、i18n 落地規格（CJK/RTL/字型需求） |

#### Domain Team（遊戲類型專家，13 個，按需啟用）

| Agent | 工具 | 產出 |
|-------|------|------|
| slot-game-expert | read, write | 老虎機數學模型、RNG 指引、GLI 認證合規、負責任遊戲設計（見「Slot Game Expert 詳解」） |
| fish-game-expert | read, write | 魚機命中機率、賠付經濟、RTP、伺服器判定 RNG、合規 |
| shooter-expert | read, write | 武器數值/彈道、命中判定、TTK 平衡、敵人 AI、手感 |
| mmo-expert | read, write | netcode、伺服器權威、狀態同步、持久化、防作弊、scope 界定 |
| rpg-systems-expert | read, write | 屬性/等級曲線、技能樹、裝備/掉落表、傷害公式 |
| card-game-expert | read, write | 卡牌數值、資源曲線、archetype/combo、平衡準則 |
| puzzle-match3-expert | read, write | board 生成/可解性、消除連鎖規則、關卡難度曲線、步數經濟 |
| platformer-expert | read, write | 跳躍手感（coyote/jump buffer）、移動物理、關卡節奏、metroidvania gating |
| roguelike-expert | read, write | 程序生成規則、build/synergy 平衡、風險報酬、meta 進度 |
| strategy-expert | read, write | 兵種相剋、資源經濟、AI 對手、塔防波次/塔數值曲線 |
| simulation-expert | read, write | 生產鏈/合成樹、供需經濟收斂、生存需求、自動化曲線 |
| rhythm-expert | read, write | 譜面設計、判定窗（ms）、延遲校正、計分/連段（綁 audio-team） |
| narrative-adventure-expert | read, write | 分支敘事結構、旗標/狀態變數、對話樹、選擇後果（綁 localization-team） |

#### Art Team（Art Lead + 5 個）

| Agent | 工具 | 產出 |
|-------|------|------|
| comfyui-team | `@comfyui`, read, write | 概念圖、PBR 貼圖、Sprite、Workflow 組裝，見「ComfyUI MCP 整合詳解」 |
| blender-team | @blender-mcp, read, write | 3D 模型 + UV、Collider Mesh、套貼圖、匯出 .fbx |
| animator | @blender-mcp, read, write | 骨骼綁定、蒙皮權重、動畫 clip、含動畫匯出（接 blender-team 的靜態 mesh） |
| vfx-artist | @comfyui, read, write | 特效素材/序列幀生成；與 technical-artist 分工：內容 vs 技術實現 |
| technical-artist | @blender-mcp, read, write, shell | Shader/材質、LOD/貼圖壓縮/合批、VFX 技術、匯入管線 |
| audio-team | `@comfyui`, read, write | SFX、BGM、配音（voice）；透過 ComfyUI `generate_audio` 生成 |

> `concept-artist` / `texture-artist` 這兩個原願景角色已合併進 `comfyui-team`，不再分別建立，因為兩者都依賴同一個 ComfyUI 工具，拆開建立沒有實際差異。
>
> 同理，原願景中 `ux-designer`（Design Team）與 `ui-artist`（Art Team）已合併成 `design/ui-ux-team`，因為兩者都圍繞 Figma 運作、共同負責 UI/UX 層。分工上：`ui-ux-team` 出版面與 Design Token（介面「怎麼排、怎麼流動」），`comfyui-team` 生成要放進版面的像素素材（icon/按鈕/背景），引擎 Team 負責在原生 UI 系統實作。
>
> 原願景的 `sound-designer` + `composer` 已合併成單一 `audio-team`（兩者都用同一個 comfyui MCP 的音訊生成能力，拆開沒有實際差異，比照 comfyui-team 的合併邏輯）。

#### Engineering Team（4 引擎 Team + Systems/UI Programmer + DevOps）

| Agent | 工具 | 產出 |
|-------|------|------|
| unity-team | `@unity-mcp`, read, write, shell | 場景組裝、遊戲邏輯、狀態機、技能系統、Build，見「Unity MCP 整合詳解」（[unity-mcp](https://github.com/CoplayDev/unity-mcp)） |
| godot-team | `@godot-mcp`, read, write, shell | 場景組裝、GDScript、State Machine、Export，見「Godot MCP 整合詳解」（[Coding-Solo/godot-mcp](https://github.com/Coding-Solo/godot-mcp)） |
| unreal-team | `@unreal-engine`, read, write, shell | 關卡組裝、Blueprint 邏輯、材質工作流程，見「Unreal MCP 整合詳解」（local MCP from [flopperam/unreal-engine-mcp](https://github.com/flopperam/unreal-engine-mcp)） |
| cocos-team | `@cocos-creator`, read, write, shell | 場景組裝、TypeScript 元件、Prefab、Build，見「Cocos MCP 整合詳解」（[cocos-mcp-server](https://github.com/DaxianLee/cocos-mcp-server)） |
| systems-programmer | read, write, shell | 引擎無關的存檔系統/資源管理/事件系統設計，交對應引擎 Team 落地實作 |
| ui-programmer | read, write, shell | 把 ui-ux-team 版面/Token 綁定成可互動引擎 UI，接 localization-team 多語落地 |
| devops-team | read, write, shell | headless build、CI pipeline、版本/產物管理、build 健康驗證 |

#### QA Team（QA Lead + 4 個）

| Agent | 工具 | 產出 |
|-------|------|------|
| functional-tester | read, shell | Unit/Integration Test、Bug 報告（驗「功能對不對」；需目標專案已有測試框架） |
| balance-tester | read, write, shell | RTP/經濟 Monte Carlo 模擬、平衡性報告（驗「數值對不對」） |
| performance-tester | read, write, shell | FPS/frame time/draw call/記憶體/載入 profiling、瓶頸分析、優化建議 |
| usability-tester | read, write | 新手引導評估、卡關點分析；與 functional/balance/performance tester 分工：驗體驗好不好 |

#### Publishing Team（2 個）

| Agent | 工具 | 產出 |
|-------|------|------|
| compliance-release | read, write, web | 分級（IARC/ESRB/PEGI）、隱私合規（GDPR/COPPA）、商店素材規格、送審清單、老虎機認證/牌照流程 |
| marketing-team | read, write | 商店文案、預告片腳本、新聞稿、社群貼文草稿、展會素材文案（純文字產出，不執行實際發布/投放，見 `docs/audio-pipeline.md` 同類誠實聲明慣例） |

---

## Agent 定義格式


每個 Agent 是 `.kiro/agents/` 下的 Markdown 檔案。YAML frontmatter 定義權限，文件本體是 system prompt。

以下是本專案**實際使用**的兩個範例（非虛構樣板）：

> ⚠️ **維護原則**：以下範例過去以「節錄貼上」的方式呈現，但實務上導致範例內容和真實檔案不同步（例如 Team 重構後範例還停留在舊版）。為避免此問題再度發生，這裡改為只列出**檔案路徑 + 核心設計原則摘要**，實際內容請直接開啟檔案查看，不在 README 裡重複貼一份可能過時的複本。

### `.kiro/agents/art/blender-team.md`

核心設計：Agent 沒有背景常駐機制，每次被選中才算「被喚醒」一次。被喚醒後第一步永遠是先判斷情境（打招呼 vs 明確需求 vs Blender MCP 未連線），再決定要不要動手，而不是預設收到訊息就開始建模。啟動時先用 `get_blendfile_summary_path_info` 做連線自檢，連不上就停止並回報。

> 「待命」不是背景常駐機制，而是 Agent 被喚醒時的第一步判斷邏輯。Kiro 的 Custom Agent 沒有 daemon；「平時待命，有需求才動」是寫進 system prompt 的行為規則，不是外部排程實現的。這條原則貫穿本專案所有已建立的 Agent。

### `.kiro/agents/orchestration/producer.md`

核心設計：Producer 不直接畫圖、不直接建模、不直接寫程式碼，只負責拆解需求、建立 Contract、用 Kiro 原生 subagent 委派給正確的 Team，並在流程尾端負責 Git commit（先 `git status` 給使用者確認，再 commit，不自動 push）。委派用扁平 `name`（`Use the "<name>" subagent to …`），並把完整 Contract 寫進 Prompt；即便如此，它不會虛構其他 Team 的執行結果或進度，只回報 subagent 真正回傳的內容。

> 寧可誠實承認限制，也不讓 Agent 表演出它做不到的能力。這條規則貫穿本專案所有已建立的 Agent（見各 Agent 檔案中的「⚠️ 現況」或「限制」章節）。


---

## 模型指派（每個 Agent 用哪個模型）


Kiro 每個 Custom Agent 可在 frontmatter 用 `model` 欄位指定模型（見「Agent 定義格式」）。以下指派的**依據是 Kiro 官方對各模型的定位**（[kiro.dev/docs/models](https://kiro.dev/docs/models/)）**＋ credit 成本倍率**（以 Auto = 1.0x 為基準，數字取自 Kiro 內 `/model` 清單與官方模型頁）。

> ⚠️ **誠實聲明**：以下是「依官方定位 × 任務性質 × 成本」推導的**合理預設**，不是本專案跑分實測的結果。請依你的實際體感調整；覺得某個 agent 產出太淺，就往上換更強的模型或調高 reasoning effort。

### 可用模型（節錄官方定位，來源：kiro.dev/docs/models）

| 模型 | credit | Kiro 官方定位（節錄） | 狀態 |
|------|--------|----------------------|------|
| `claude-opus-4.8` | 2.2x | 「最高可靠度：最強自我驗證，會標記不確定、證據不足時反駁，而非硬報進度」 | Active |
| `claude-sonnet-5` | 1.3x | 「接近 Opus 4.8 的推理與工具使用，動手前先規劃、能自主跑完多步 agentic 任務」 | Experimental |
| `claude-haiku-4.5` | 0.4x | 「接近前沿智慧、1/3 成本，適合快速迭代與 subagent」 | Active |
| `glm-5` | 0.5x | 「repo 規模的長流程 agentic，200K context，跨檔案遷移／全端開發」（開源權重） | Experimental |
| `minimax-m2.5` | 0.25x | 「接近 Opus 等級的 coding，0.25x 成本，涵蓋完整開發生命週期」（開源權重） | Experimental |
| `auto` | 1.0x | 「Kiro 自動路由，依任務挑最佳模型，平衡品質與成本」 | Active |

> 其他可選：`deepseek-3.2`（0.25x，agentic/多步推理）、`minimax-m2.1`（0.15x，多程式語言）、`qwen3-coder-next`（**0.05x 最省**，256K，長時 coding）、Claude Opus 4.5–4.7 / Sonnet 4.x。完整清單以你環境的 `/model` 為準。

### 本專案的指派

| Agent | 模型 | credit | 依據（對應官方定位） |
|-------|------|--------|---------------------|
| `slot-game-expert` | `claude-opus-4.8` | 2.2x | 數學模型/RTP/認證錯誤代價最高 → Opus 4.8「最高可靠度、會標記不確定、~4x 更少讓程式碼瑕疵溜過」 |
| `fish-game-expert` | `claude-opus-4.8` | 2.2x | casino 數學/RTP/合規，與 slot 同等級正確性要求 → Opus 4.8 |
| `shooter-expert` | `claude-sonnet-5` | 1.3x | 武器/命中/AI 系統設計推理 → Sonnet 5 |
| `mmo-expert` | `claude-sonnet-5` | 1.3x | netcode/跨系統架構設計推理 → Sonnet 5 |
| `rpg-systems-expert` | `claude-sonnet-5` | 1.3x | 數值/公式/系統設計 → Sonnet 5 |
| `card-game-expert` | `claude-sonnet-5` | 1.3x | 卡牌數值/平衡設計 → Sonnet 5 |
| `puzzle-match3-expert` | `claude-sonnet-5` | 1.3x | board 可解性/難度曲線/步數經濟設計 → Sonnet 5 |
| `platformer-expert` | `claude-sonnet-5` | 1.3x | 跳躍手感參數/關卡節奏/gating 設計 → Sonnet 5 |
| `roguelike-expert` | `claude-sonnet-5` | 1.3x | 程序生成/build synergy/meta 進度設計 → Sonnet 5 |
| `strategy-expert` | `claude-sonnet-5` | 1.3x | 兵種相剋/經濟/AI/波次曲線設計 → Sonnet 5 |
| `simulation-expert` | `claude-sonnet-5` | 1.3x | 生產鏈/供需經濟收斂設計 → Sonnet 5 |
| `rhythm-expert` | `claude-sonnet-5` | 1.3x | 譜面/判定窗/延遲校正設計 → Sonnet 5 |
| `narrative-adventure-expert` | `claude-sonnet-5` | 1.3x | 分支敘事/旗標/對話樹結構設計 → Sonnet 5 |
| `creative-director` | `claude-sonnet-5` | 1.3x | 願景仲裁/創意判斷，需結構化推理 → Sonnet 5 |
| `producer` | `claude-sonnet-5` | 1.3x | 拆任務/調度/串接多步 → Sonnet 5「接近 Opus 工具使用、能跑完多步 agentic」，比 Opus 省 |
| `game-designer` | `claude-sonnet-5` | 1.3x | spec 導向文件 → Sonnet 5「適合 spec 導向、高保真實作」 |
| `design-lead` | `claude-sonnet-5` | 1.3x | 整合/審查設計規格、消矛盾 → Sonnet 5 |
| `economy-designer` | `claude-sonnet-5` | 1.3x | 經濟數值設計，需結構化推理 |
| `balance-tester` | `claude-sonnet-5` | 1.3x | 寫模擬程式 + 統計判讀，多步 agentic |
| `qa-lead` | `claude-sonnet-5` | 1.3x | 測試策略/彙整/go-no-go 判斷 → Sonnet 5 |
| `performance-tester` | `claude-sonnet-5` | 1.3x | profiling 判讀 + 量測腳本，多步 → Sonnet 5 |
| `unity/godot/unreal/cocos-team` | `claude-sonnet-5` | 1.3x | 寫程式 + 大量 MCP 工具編排，要高工具可靠度 → Sonnet 5「近 Opus 工具使用」 |
| `tech-lead` | `claude-sonnet-5` | 1.3x | 架構決策/跨引擎 code-review，需高可靠度 → Sonnet 5 |
| `devops-team` | `claude-sonnet-5` | 1.3x | CI/build 腳本，需正確性 |
| `blender-team` / `animator` | `glm-5` | 0.5x | 寫 Blender Python（程式）但非關鍵 → GLM-5「長流程 agentic coding、跨檔案」，成本減半 |
| `art-lead` / `technical-artist` | `glm-5` | 0.5x | 風格審查 / shader / 優化，美術向 coding → GLM-5，成本減半 |
| `ui-ux-team` | `glm-5` | 0.5x | Figma + handoff 規格，generalist 夠用、省成本 |
| `compliance-release` | `glm-5` | 0.5x | 查政策 + 條列清單，doc/結構化為主 |
| `marketing-team` | `glm-5` | 0.5x | 文案/腳本寫作，generalist 夠用、省成本 |
| `comfyui-team` / `audio-team` | `minimax-m2.5` | 0.25x | 主要在驅動 MCP 工具 → MiniMax M2.5「接近 Opus 的 coding、0.25x」，高 CP 值 |
| `localization-team` | `minimax-m2.5` | 0.25x | 抽字串/locale 偏機械性，成本優先 |
| `functional-tester` | `claude-haiku-4.5` | 0.4x | 跑測試回報 → Haiku 4.5「接近前沿、1/3 成本、適合 subagent」 |

### 調整槓桿

- **想更省**：engine team 可改 `glm-5`（0.5x，官方定位「repo 規模 agentic、跨檔案」）；`minimax` 那批可降到 `qwen3-coder-next`（0.05x）。
- **想更穩**：關鍵 agent 升到 `claude-opus-4.8`，或在 chat 把 reasoning effort 調到 High/Max（官方：effort 越高越深入但更耗 credit）。
- **懶得逐一調**：全設 `auto`（1.0x），Kiro 自動路由。
- **怎麼改**：編輯各 agent frontmatter 的 `model`，值必須是你 `/model` 清單裡的**確切 ID**；填了不存在的 ID 會自動退回預設模型 + 警告。
- **注意（Experimental 與 region，會影響可用性）**：依 [kiro.dev/docs/models](https://kiro.dev/docs/models/)（2026-07-01）——
  - `claude-sonnet-5`（本專案約 20 個 agent 的預設）與 `glm-5`：**Experimental，且僅在 us-east-1**。
  - `minimax-m2.5`：Experimental，us-east-1 / eu-central-1 皆可。
  - `claude-haiku-4.5`、`claude-opus-4.8`：Active，雙區皆可。
  - 影響：若你的 Kiro 未路由到 us-east-1，占多數的 Sonnet 5 / GLM-5 agent 可能不可用。屆時可把預設換成 Active 且雙區的 `claude-sonnet-4.6`，或直接用 `auto` 讓 Kiro 自動路由。
