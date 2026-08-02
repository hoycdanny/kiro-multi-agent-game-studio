# Kiro Multi-Agent Game Studio

[English](README.md) | [繁體中文](README_ZH.md) | [简体中文](README_CN.md) | [日本語](README_JP.md) | [한국어](README_KR.md)

> **語言版本說明**：為了服務全球社群，README 提供 5 種語言版本。功能說明、範例與 demo 步驟在所有語言版本之間刻意保持平行一致，每個版本都經過該語言的自然語感審閱。`docs/` 下的深度參考文件與 `.kiro/agents/` 下的 Agent 定義是繁體中文，並在每個檔案開頭附上英文摘要區塊，沿用 [kiro-unity-accelerator](https://github.com/hoycdanny/kiro-unity-accelerator) 的同一套慣例。每個 Agent 都會用你使用的語言回應——內部檔案是繁體中文並不會限制對話語言。若你遇到任何語言障礙，請開一則 issue 告訴我們。

把你的 IDE 變成一間虛擬遊戲工作室。用日常語言描述你想做什麼遊戲，一支協同運作的 **48 個 AI Agent** 團隊——Producer、五位 Lead、遊戲類型專家、美術、引擎團隊、QA、發行——會替你規劃、實作，並透過明確的 Contract 在彼此之間交付產出。

領域知識不住在這個 repo 裡。它住在 **29 個機器層級安裝的 [Kiro Power](https://kiro.dev/docs/powers/)** 裡，每一個都獨立維護、並對真實工具連線驗證過。這個 repo 只放**組織層**：誰做什麼、依什麼順序做、交付什麼東西。

> **為什麼要分兩層**：手抄進 Agent prompt 裡的工具知識會過時。在這次分層之前，`unity-team.md` 裡有 7 個已經不存在的 API 呼叫。Power 是對真實連線驗證過、而且獨立更新的，所以 Agent prompt 只需要承載角色定位與交接紀律。詳見 [docs/powers-inventory.md](docs/powers-inventory.md)。

> **核心概念**：本文件通篇會用到的術語（你不需要一開始就全部搞懂）：
> - **Agent**：一份角色定義（`.kiro/agents/*.md`），有自己的 system prompt、模型與工具權限
> - **Power**：一個 [Kiro Power](https://kiro.dev/docs/powers/)——打包好的領域知識層（Steering 檔案）加上選配的 MCP server，安裝在機器層級的 `~/.kiro/powers/` 底下
> - **MCP**（Model Context Protocol）：一種標準化協定，讓 AI 助手能用自然語言操作開發工具——Unity、Blender、ComfyUI、Figma 等等
> - **Steering**：Power 或專案注入 Agent context 的 markdown 知識檔案，可以是每次都載入，也可以是條件式載入
> - **Contract**：Agent 之間互相交接工作所用的 YAML 交接格式（Task Contract／Asset Contract／Change Request）
> - **Subagent 委派**：Producer 分派工作的方式——每個 Subagent 都跑在隔離的 context window 裡，所以完整的 Contract 必須寫進委派 prompt

## 功能特色

- **單一入口** — 跟 `producer` 說話；它會偵測你的引擎與遊戲類型，再分派給對的 Lead 與 Specialist。你不需要知道任何一個 Agent 的名字。
- **4 種引擎** — Unity、Godot、Unreal、Cocos Creator。Producer 會分派到對應的引擎團隊，而不是預設只有一種。
- **13 類遊戲類型** — 老虎機、魚機、射擊、MMO、RPG、卡牌、三消、平台跳躍、roguelike、策略、模擬、節奏、敘事冒險。每一類都有專屬的 Domain Expert，背後掛著對應的 Power。
- **諮詢模式** — 你只要說「我不懂遊戲」，Lead 就會給你一個帶理由、帶取捨、帶預設值的建議讓你往前走，而不是丟一串技術問題把你擋在門外。
- **外部化的知識** — 29 個 Power、323 個 Steering 檔案、約 4.9 MB 的領域知識，全部在這個 repo 之外、可獨立更新。
- **量化過的領域知識** — Power 把設計問題變成數學：整數除法造成的 TTK 斷崖、掉落率的長尾（P90 = 平均值的 2.3 倍）、從跳躍高度與頂點時間反推的跳躍物理、MMO 的 T1–T4 範圍分級。
- **明確的 Contract** — 每一次交接都是一份帶驗收條件的 YAML Contract；每一次交付都會寫一則 Manifest，讓下游 Agent 知道產出了什麼、還有什麼是壞的。
- **誠實的能力邊界** — 每個 Power 都會聲明自己無法驗證什麼。Agent 會停下來回報知識缺口，而不是憑印象猜工具 API。
- **信心等級** — 領域事實會標記為 `HIGH`（可推導）、`MEDIUM`（慣例）或 `UNVERIFIED`（產業數字，需要你用自家數據校準）。Agent 會照實轉述等級，而不是把所有數字都講得一樣可信。

## 架構

```
                    You
                     │
              ┌──────▼──────┐
              │  producer   │  detects engine + genre, builds contracts
              └──────┬──────┘
        ┌────────┬───┴────┬─────────┬─────────┐
        ▼        ▼        ▼         ▼         ▼
   design-   domain-   art-      tech-     qa-lead     ← 5 leads (L2)
    lead      lead     lead      lead                    no Power by design
        │        │        │         │         │
        ▼        ▼        ▼         ▼         ▼
    40 specialists (L3) ── each reads its Power before acting
        │
        ▼
   shared/ artifacts  +  .kiro/state/handoffs/*.delivery.yaml
```

有三個結構性決定值得先知道：

**Power 只在 Specialist 層被讀取。** Producer 與五位 Lead 都沒有 Power。Lead 的價值在於跨 Specialist 的取捨判斷——你不可能問 `unity-team` 該不該用 Unity，因為它永遠會說該。給 Lead 掛上 Power 會讓它偏向那個領域，正好毀掉它存在的意義。

**Agent 之間靠檔案溝通，不靠對話。** Subagent 跑在隔離的 context 裡，所以彼此之間沒有即時通道。設計真相住在 `.kiro/steering/project/gdd.md`，交付物住在 `shared/`，交接回執住在 `.kiro/state/handoffs/`。

**Producer 就是那個路由器。** 它讀上游的 Delivery Manifest，把內容寫進下一個 Agent 的委派 prompt。沒有任何東西是被假設為隱含共享的。

完整的資料流、治理方式與功能開發生命週期：[docs/architecture-and-process.md](docs/architecture-and-process.md)。

## 前置需求

| 需求 | 說明 |
|-------------|-------|
| [Kiro IDE](https://kiro.dev/) | Agent、Power 與 Steering 全部由 Kiro 載入 |
| Git + [Git LFS](https://git-lfs.com/) | 二進位資產透過 LFS 追蹤（見 `.gitattributes`） |
| [uv](https://docs.astral.sh/uv/) | Blender、ComfyUI 與 Unreal 的 MCP server 需要它 |
| 你的目標引擎 | Unity / Godot / Unreal / Cocos Creator——只裝你真的會用的那一個 |
| Node.js | 只有在使用 Godot MCP server 時需要（透過 `npx` 安裝） |

依 Pipeline 選配：Blender（3D）、ComfyUI（2D 生成）、Krita（手繪美術）、Ableton Live（音樂）、一個 Figma 帳號（UI）。

## 安裝

### 步驟一 — Clone 並啟用 LFS

```bash
git clone https://github.com/hoycdanny/kiro-multi-agent-game-studio.git
cd kiro-multi-agent-game-studio
git lfs install    # 每台機器做一次；沒裝的話先 brew install git-lfs
```

### 步驟二 — 在 Kiro 中開啟並信任這個 workspace

用 Kiro IDE 開啟這個資料夾。第一次開啟時它會問你是否信任這個 workspace——**選擇信任**，否則 Agent 與 Steering 都不會載入。接著 Agent Selector 就會列出全部 48 個 Agent。

### 步驟三 — 安裝你需要的 Power

Kiro → Powers 面板 → **Add custom power** → 來源填 `https://github.com/hoycdanny/<power-name>`。

**你不需要裝滿 29 個。** 只裝這個專案會用到的——某個 Agent 找不到自己的 Power 時會誠實回報缺口，不會硬掰。

任何遊戲都用得上的最小組合：

```
kiro-<your-engine>-accelerator    unity / godot / unreal / cocos — 選一個
kiro-comfyui-accelerator          2D 資產生成（幾乎一定會用到）
kiro-economy-balancing-expert     經濟數值 + balance-tester 依賴的模擬方法論
kiro-game-compliance-expert       一旦你打算出貨就會需要
```

依需要加裝：

| 如果你要做 | 安裝 |
|------------------|---------|
| 3D 模型／動畫 | `kiro-blender-accelerator` |
| 手繪 UI 或 sprite | `kiro-krita-accelerator` |
| 原創音樂 | `kiro-ableton-accelerator` |
| Figma 設計 → 引擎 UI | `figma`（Kiro 官方推薦清單裡的，不是 `hoycdanny` 的） |
| 老虎機／魚機 | `kiro-slot-game-expert` / `kiro-fish-game-expert` |
| 錢包或金流後端 | `kiro-gaming-wallet-expert` |
| RPG／射擊／卡牌／三消／平台跳躍／節奏／策略／模擬／roguelike／敘事 | 對應的 `kiro-<genre>-expert` |
| 多人連線 | `kiro-mmo-netcode-expert`——**先讀它的 T1–T4 範圍分級；大多數專案並不需要真正的 MMO** |
| 存檔系統／資源管理 | `kiro-game-systems-expert` |
| 多語系 | `kiro-i18n-expert` |
| CI／自動化建置 | `kiro-game-devops-expert` |
| 可用性評估 | `kiro-usability-expert` |

驗證：

```bash
ls ~/.kiro/powers/installed/                                        # 已安裝的 Power
ls ~/.kiro/powers/installed/kiro-blender-accelerator/steering/      # 它的 steering 檔案
ls ~/.kiro/powers/repos/kiro-blender-accelerator/templates/         # templates 在 repos/ 底下，不在 installed/
```

### 步驟四 — 連上你的工具 MCP server

`.kiro/settings/mcp.json` 已經包含 `blender-mcp`、`comfyui`、`unity-mcp`、`godot-mcp`、`unreal-engine`、`cocos-creator`、`figma` 與 `github` 的設定。

> ⚠️ **`ableton` 與 `krita` 還不在 `mcp.json` 裡。** 若你需要音樂或手繪美術 Pipeline，請手動加上——設定內容在 [docs/mcp-integrations.md](docs/mcp-integrations.md)。

接著啟動你真的會用到的工具：

| 工具 | 怎麼連 |
|------|----------------|
| Blender | 啟用 `blender_mcp` 外掛並啟動它的 server（預設 `localhost:9876`） |
| ComfyUI | 啟動本機服務 |
| Unity | Window → MCP for Unity → Start Server |
| Godot | `npx` 會自動安裝 `@coding-solo/godot-mcp`；記得設定 `GODOT_PATH` |
| Unreal | 安裝 UnrealMCP 外掛並在 Editor 中啟用 |
| Cocos Creator | 安裝 `cocos-mcp-server`，然後 Extension → Cocos MCP Server → Start |
| Figma | 官方 Remote MCP Server；第一次使用時在 Kiro 中完成 OAuth |

各工具的細節、疑難排解與其他連線模式：[docs/mcp-integrations.md](docs/mcp-integrations.md)。

## 使用方式

### 三種入口

| 你的情況 | 找誰 | 為什麼 |
|----------------|---------|-----|
| 你有目標但沒有遊戲開發背景 | `producer` | 它會進入諮詢模式並委派 Lead 給建議，而不是反過來拷問你 |
| 你懂這個領域，想要一個專業判斷 | 對應的 **Lead** | 少一次分派轉手；Lead 會直接回答選型問題 |
| 範圍很窄、可獨立回答的問題 | 對應的 **Specialist** | 例如直接問 `shooter-expert` TTK 怎麼算 |

各位 Lead 能替你決定什麼：

| Lead | 能決定 |
|------|---------|
| `tech-lead` | **引擎選型**、架構取捨、效能預算、你到底需不需要多人連線 |
| `domain-lead` | 這是哪一種遊戲類型、該啟用哪位 Domain Expert、多類型重疊時的主從關係 |
| `design-lead` | 核心循環該長什麼樣、範圍該切多小、先做哪個系統 |
| `art-lead` | 美術方向、2D 或 3D、生成式與手繪的分工、聲音基調 |
| `qa-lead` | 這個階段該測什麼、什麼程度算可以出貨 |

### 範例指令

```
「用 Unity 做一台老虎機」
「我想做老虎機，但我對遊戲一無所知」                             → 進入諮詢模式
「手機三消遊戲我該用哪個引擎？」                                → 問 tech-lead
「血量 100、傷害 33，TTK 是多少？」                             → 問 shooter-expert
「40 張牌的牌組放 3 張同名卡，開場 5 張抽到的機率是多少？」       → 問 card-game-expert
「在 Unity 實作這棵技能樹，規格在 docs/skill-tree-spec.md」      → 跳過諮詢模式
```

### Demo：「我想做老虎機，但我不懂遊戲」

1. **`producer`** 偵測到兩件事：類型是 casino，而且使用者自己聲明沒有背景 → 進入**諮詢模式**（`.kiro/steering/global/advisory-mode.md`）。

2. 它**不會**丟一堆技術問題給你。它委派 `tech-lead` 做引擎選型，委派 `domain-lead` 確認該啟用哪位專家。

3. **`tech-lead`** 用四段式諮詢格式回答：
   > **建議**：Cocos Creator。
   > **理由**：老虎機是 2D，需要 web 加手機兩個目標平台，而且動畫與 UI 吃重；這個組合下，Cocos 的 2D 管線最直接、web 匯出的成熟度也最高。
   > **取捨**：如果你之後想做 3D 版本，或你手上已經有 Unity 的人力，Unity 更好；純 web 前端團隊也可以考慮 PixiJS。
   > **預設值**：如果你不回覆，就用 Cocos Creator 往前走。

4. **`slot-game-expert`** 讀 `kiro-slot-game-expert`，並且**先問你的目標司法管轄區**——因為「最小 spin 間隔該設多少」在不同市場有不同的法律答案。如果你說還沒決定，它會用最保守的假設往前走（純娛樂原型、不涉及真實金錢），並明確標明這個假設。

5. Producer 轉述這些建議，然後只問**一個**問題：「這樣可以開始嗎？」

6. 你同意之後，Pipeline 就跑起來：

```
slot-game-expert   → 數學模型（RTP / 波動度 / 賠付表）
balance-tester     → 讀 simulation-methodology.md，用 Monte Carlo 驗證實際 RTP
art-lead           → comfyui-team 生成符號與背景
ui-ux-team         → 讀 figma Power，產出版面 + Design Token
cocos-team         → 讀 kiro-cocos-accelerator，組裝場景與邏輯
qa-lead            → functional-tester 驗證流程
compliance-release → 讀 kiro-game-compliance-expert（如果你打算出貨）
```

你總共只回答了一次「好」。這就是諮詢模式的意義。

另外兩個完整走法（你已經有規格的情況；純分析、不產出任何檔案的情況）以及查看專案狀態的檔案地圖：[docs/orchestration-guide.md](docs/orchestration-guide.md)。

## 專案結構

```
.kiro/
├── agents/              48 個 Agent 定義，依 layer 分組
│   ├── orchestration/   creative-director, producer
│   ├── design/          5 個核心設計角色 + 13 個類型 Domain Expert + ui-ux
│   ├── art/             blender, comfyui, krita, animator, audio, vfx, technical-artist
│   ├── engineering/     4 個引擎團隊 + systems/ui programmer, devops, wallet
│   ├── qa/              functional / balance / performance / usability + qa-lead
│   └── publishing/      compliance-release, marketing-team
├── steering/
│   ├── global/          contracts, asset-standards, bug-severity, powers-registry, advisory-mode
│   └── project/         gdd, style-guide, milestones      ← 你這款遊戲的單一真相來源
├── state/               tasks.yaml, handoffs/*.delivery.yaml
└── settings/mcp.json    MCP server 設定

shared/                  跨 Agent 的產出中轉站
├── concept/ textures/ sprites/ ui/     來自 comfyui-team
├── models/                             來自 blender-team
├── rigs/ animations/                   來自 animator
├── audio/{sfx,music,voice}/            來自 audio-team
├── locales/                            來自 localization-team
└── sim/                                來自 balance-tester

docs/                    參考文件（繁體中文 + 英文摘要）
```

`~/.kiro/powers/` — 知識層，**在這個 repo 之外**，機器層級安裝。

## Agent 分層

| 層級 | 數量 | 組成 |
|-------|:-----:|-------------|
| L0 戰略 | 2 | `creative-director`（願景守門）、`producer`（分派中樞） |
| L2 Lead | 5 | `design` / `domain` / `art` / `tech` / `qa`——分派中介與品質關卡，**刻意不掛 Power** |
| L3 設計與類型 | 20 | 7 個核心設計角色 + 13 個類型 Domain Expert |
| L3 美術與聲音 | 7 | Blender、ComfyUI、Krita、Animator、Audio、VFX、Technical Artist |
| L3 工程 | 8 | 4 個引擎團隊 + Systems/UI Programmer、DevOps、Wallet |
| L3 QA | 4 | functional / balance / performance / usability |
| L3 發行 | 2 | compliance-release、marketing-team |

**48 個裡有 33 個掛了 Power**；另外 15 個是協調角色，它們的知識*就是*本專案的組織慣例。完整清單、每個未掛 Power 的 Agent 的理由，以及覆蓋缺口分析：[docs/powers-inventory.md](docs/powers-inventory.md)。

## 疑難排解

| 症狀 | 原因 | 解法 |
|---------|-------|-----|
| Agent 回報找不到某個 Power 的 steering | Power 沒安裝 | Kiro → Powers 面板 → 安裝 `hoycdanny/<power-name>`；用 `ls ~/.kiro/powers/installed/` 驗證 |
| Agent 丟出一連串技術問題 | 諮詢模式沒被觸發 | 明確說：「我不懂這塊——給我一個建議和一個預設值」 |
| Agent 呼叫了不存在的 MCP 工具 | 沒有遵守 Steering-First | 叫它先讀對應 Power 的 steering 再操作。**這是已知弱點——見下方說明** |
| 兩位 Specialist 給出互相矛盾的數字 | 缺少 Lead 的整合 | 回到 Producer，請它委派相關 Lead 做一次整合審查 |
| 產出落在奇怪的地方 | 沒讀 `asset-standards.md` | 指出正確的目標目錄（`shared/<type>/`）與命名規則 |
| Beta 之後有人想加新功能 | 沒有提出 Change Request | 請 Producer 產出一份 CR（`contracts.md`）；只有你核准後才會執行 |
| Agent 說「應該沒問題」但沒有任何證據 | 沒有遵守驗證紀律 | 要求可查核的數字——每個 Power 的 `verification-policy.md` 都明訂必須附上什麼 |
| 某位 Lead 回報它無法委派 | 巢狀委派的限制 | 請 Producer 直接分派那位 Specialist（這是有記錄的退化策略） |

各工具的症狀對照原因表在 [docs/mcp-integrations.md](docs/mcp-integrations.md)；調度層級的對照表在 [docs/orchestration-guide.md](docs/orchestration-guide.md)。

## 已知限制

這些是架構性的，不是 bug。知道它們可以避免意外。

**Steering-First 沒有機制強制。** Power 內含 `hooks/pre-*-tool.json`（preToolUse 防護，用意是在任何工具呼叫前強迫先讀 steering），但依 Kiro 的官方文件，**subagent 不會觸發 Hooks**——而這個專案的整條 Pipeline 都跑在 subagent 委派上。那道防護在這裡完全不生效。這正是當初讓 `unity-team` 累積出 7 個幽靈 API 的同一個根本原因。

**兩層委派尚未完整驗證。** Kiro 的文件對巢狀 subagent 委派沒有任何保證。這個專案採用 producer → lead → specialist；若某次巢狀分派失敗，退化策略是由 Producer 直接分派該 Specialist。

**Subagent 讀不到 Specs，也不會觸發 Hooks。** `.kiro/specs/` 底下的任何東西在 subagent 內都是看不見的。不要只把關鍵規格放在那裡——放進 `gdd.md`，或直接寫進委派 prompt。

**Power 內容中有不小的比例是 `UNVERIFIED`。** 產業平均值、法規細節、引擎端的匯入行為、平台延遲數字，全都標記為需要你用自家數據校準。如果你看到一個具體數字卻沒有等級標記，請追問它是可推導的、還是需要校準的。

**這裡沒有人能告訴你這款遊戲好不好玩。** 每個 Power 都在自己的能力邊界裡寫明這一點。數字可以模擬、關卡可以驗證是否走得通、效能可以對照預算量測——但手感與樂趣需要真人實測。`usability-tester` 提供的是評估框架，它**沒辦法真的去玩這款遊戲**；當你要求它跑一次可用性測試時，它會把交付標記為 `blocked`，而不是 `delivered`。

## 建議的第一步

不是把 48 個 Agent 全部跑一遍。而是：**做一款極小的遊戲，從頭走到尾，直到你拿到一個可執行的 build。**

這條 Pipeline 有很多接縫——Contract 傳遞、產出落地、Delivery Manifest、引擎匯入、建置驗證——每一道都只能靠真的用過才能證明。用一個你兩天做得完的東西把整條路徑驗證一遍，價值遠大於先寫一份詳盡的設計文件。

檢查清單在 [docs/orchestration-guide.md](docs/orchestration-guide.md#8-建議的第一步)：

- [ ] Producer 正確偵測出類型與引擎，分派給對的 Lead
- [ ] Lead 有轉發給 Specialist，並收回結果
- [ ] Specialist 真的讀了自己的 Power steering（問它引用了哪個檔案）
- [ ] 產出落在正確的 `shared/` 目錄，檔名符合規範
- [ ] 有寫出 Delivery Manifest，而且下游讀得到
- [ ] 引擎團隊匯入了上游產出，並產出一個可執行的 build
- [ ] QA 至少回報一個帶嚴重度標記的問題（驗證 `bug-severity.md` 有被遵守）

## 文件

| 文件 | 內容 |
|----------|----------|
| [docs/orchestration-guide.md](docs/orchestration-guide.md) | **使用方式從這裡開始** — 三種入口、各位 Lead 決定什麼、三個完整走法、檔案地圖、疑難排解、限制 |
| [docs/powers-inventory.md](docs/powers-inventory.md) | 全部 29 個 Power 依類型分組、為什麼 15 個 Agent 沒有掛、信心等級、覆蓋缺口分析 |
| [docs/mcp-integrations.md](docs/mcp-integrations.md) | 十個 MCP 整合（Blender / ComfyUI / Unity / Godot / Unreal / Cocos / Figma / GitHub / Ableton / Krita） |
| [docs/agents-and-roles.md](docs/agents-and-roles.md) | Domain Expert 細節、角色職責、Agent 定義格式、模型配置 |
| [docs/architecture-and-process.md](docs/architecture-and-process.md) | 工具鏈與資料流、開發流程、Contract、治理、漸進式擴充 |
| [docs/missing-powers.md](docs/missing-powers.md) | Power 建置規格（18 個全部完成）— 保留下來作為新增 Power 的範本 |
| [docs/audio-pipeline.md](docs/audio-pipeline.md) | 配音與音樂 Pipeline：AI 生成與真人製作的取捨、授權檢查清單 |
| [docs/reference.md](docs/reference.md) | 成本估算、錯誤處理與降級、設計理由、檔案結構 |
| [docs/closing-kit-checklist.md](docs/closing-kit-checklist.md) | 出貨封存檢查清單 |

所有 Agent 都會自動載入的共享規範：

| 檔案 | 用途 |
|------|---------|
| `.kiro/steering/global/contracts.md` | Task Contract／Asset Contract／Change Request 格式、委派命名、Delivery Manifest |
| `.kiro/steering/global/powers-registry.md` | Agent ↔ Power 對照、磁碟路徑、使用紀律、信心等級的轉述規則 |
| `.kiro/steering/global/advisory-mode.md` | 當你缺乏領域知識時 Lead 該怎麼表現；決定的緊急程度分級 |
| `.kiro/steering/global/asset-standards.md` | 命名、poly budget、音訊格式、產出落地目錄 |
| `.kiro/steering/global/bug-severity.md` | 四條 QA 線共用的 S1–S4 嚴重度定義 |
| `.kiro/steering/project/gdd.md` | **你這款遊戲的單一真相來源** — 概念、核心循環、系統規格、數值 |
| `.kiro/steering/project/style-guide.md` | 美術與聲音方向 |
| `.kiro/steering/project/milestones.md` | 從 Prototype 到 Gold 各階段的驗收標準 |

## 參與貢獻

見 [CONTRIBUTING.md](CONTRIBUTING.md)。這個專案歡迎新的 Agent、新的 Power，以及對過時事實的訂正——尤其是最後這一項，因為過時正是這套架構存在的目的所要對抗的失效模式。

## 安全性

絕對不要把憑證、簽章金鑰、keystore 或 API token 提交進來。`.gitignore` 涵蓋了常見的情況，但提交前還是自己看一遍 diff。若你發現安全問題，請開一則 issue，不要直接開公開的 pull request。

## 授權條款

MIT — 見 [LICENSE](LICENSE)。
