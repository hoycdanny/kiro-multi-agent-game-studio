# 調度指南：使用者怎麼驅動這座工作室

> **這份檔案是什麼**：48 個 agent、25 個 Power，實際使用時你要對誰說話、他們之間怎麼傳遞、知識在哪一層被讀取、卡住時怎麼救。這是**給使用者看的操作手冊**，不是給 agent 讀的 steering。
>
> 若你只想知道「該對誰說話」：**多數情況對 `producer` 說就好**。其餘是細節。

---

## 1. 一分鐘版本

```
你 ──▶ producer ──▶ Lead（5 個）──▶ Specialist（40 個）──▶ 讀 Power ──▶ 產出
                                                              │
     ◀── 彙整回報 ◀── 審查 ◀────────── Delivery Manifest ◀────┘
```

- **你只需要對 `producer` 描述目標**，不需要知道有哪些 agent
- Producer 判斷這是什麼類型的需求 → 委派給對應的 **Lead**
- Lead 轉發給旗下 **Specialist**，Specialist 動手前先讀自己的 **Power**
- 產出寫進 `shared/`，交接紀錄寫進 `.kiro/state/handoffs/`
- 結果沿原路彙整回你

**你不需要記任何 agent 名字。** 但知道下面這些會讓你用得更精準。

---

## 2. 三種入口，依你手上有多少資訊選

| 你的狀態 | 對誰說 | 為什麼 |
|---------|-------|-------|
| **只有目標，不懂遊戲開發** | `producer` | 它會啟動諮詢模式，委派 Lead 給你建議而不是丟問題給你 |
| **知道問題屬哪個領域，要專業判斷** | 對應的 **Lead** | 跳過一層轉手，Lead 直接給選型建議 |
| **問題很窄、已知該問誰** | 對應的 **Specialist** | 例如「TTK 怎麼算」直接問 `shooter-expert` |

### 2.1 五個 Lead 各自能替你決定什麼

這是**跳過 Producer 直接找人**時最有用的一張表：

| Lead | 能替你決定 | 典型問句 |
|------|-----------|---------|
| `tech-lead` | **引擎選型**、架構取捨、效能預算、要不要上多人連線 | 「我做老虎機該用哪個引擎」 |
| `domain-lead` | 這需求屬哪個遊戲類型、該啟用哪位 Domain Expert、多類型疊加的主從 | 「這算 roguelike 還是 RPG」 |
| `design-lead` | 核心循環長什麼樣、範圍切多小才做得完、先做哪個系統 | 「第一版該做到哪」 |
| `art-lead` | 美術風格方向、2D/3D 取捨、生成式 vs 手繪分工、聲音基調 | 「這個題材適合什麼風格」 |
| `qa-lead` | 這階段該測什麼、什麼程度算可以出貨 | 「現在可以上線嗎」 |

**為什麼選型必須問 Lead 而不是 Specialist**：你不可能問 `unity-team`「該不該用 Unity」——它必然說該。四個引擎 Team 各有立場，兩個 casino Domain Expert 也都想接案。**Lead 在它管的範圍內沒有單一工具的包袱**，這是它存在的結構理由。

### 2.2 直接找 Specialist 的時機

問題窄、答案不需要跨領域協調時，直接問最快。例如：

| 問句 | 對誰 | 它會讀的 Power |
|------|------|--------------|
| 「HP 100、傷害 33 的 TTK 是多少」 | `shooter-expert` | `kiro-shooter-expert` |
| 「1% 抽卡率要抽幾次才有 90% 機率中」 | `economy-designer` | `kiro-economy-balancing-expert` |
| 「40 張牌組放 3 張某卡，開手 5 張抽到的機率」 | `card-game-expert` | `kiro-card-game-expert` |
| 「這個 FBX 匯進 Unity 比例不對」 | `blender-team` | `kiro-blender-accelerator` |
| 「跳躍高度 3 格、0.35 秒到頂，重力設多少」 | `platformer-expert` | `kiro-platformer-expert` |

**注意**：Specialist 不會幫你協調下游。它給你規格，但不會自動叫引擎 Team 去實作——那要回到 Producer。

---

## 3. 情境走查：三個真實流程

### 情境 A：完全新手，只有一句話

> **你**：我想做一個老虎機，但我不懂遊戲開發。

**會發生什麼**：

1. **`producer`** 偵測到兩件事：類型是老虎機（casino）、使用者聲明不懂 → **進入諮詢模式**（`advisory-mode.md`）

2. Producer **不會**丟一串技術問題給你。它委派：
   - `tech-lead` → 引擎選型建議
   - `domain-lead` → 確認該啟用 `slot-game-expert`

3. **`tech-lead`** 回覆的格式是四段（諮詢模式規定）：
   > **建議**：Cocos Creator。
   > **理由**：老虎機是 2D、需要跨 web 與行動平台、動畫與 UI 密集；Cocos 的 2D 管線與 web 匯出成熟度對這個組合最直接。
   > **取捨**：若你之後要做 3D 場景版本或已有 Unity 團隊，改用 Unity 更好；純 web 前端團隊可考慮 PixiJS。
   > **預設值**：不回應就用 Cocos Creator 往前走。

4. **`domain-lead`** 啟用 `slot-game-expert`，它讀 `kiro-slot-game-expert` 的 steering。**這裡是關鍵**：它會先問司法管轄區，因為「最小旋轉間隔設多少」在不同市場答案不同（Power 的 `advisory-engagement.md` 明訂「不問市場無法正確回答」）。若你說「還沒決定」，它會用**最保守假設**（純娛樂原型、不涉真實金錢）往前走並標明這個假設。

5. Producer 把建議彙整回你，**只問一句**：「這樣可以開始嗎？」

6. 你同意後，Pipeline 展開：
   ```
   slot-game-expert  → 數學模型（RTP／波動度／paytable）
   balance-tester    → 讀 kiro-economy-balancing-expert 的 simulation-methodology.md 跑模擬驗 RTP
   art-lead          → comfyui-team 產符號與背景
   ui-ux-team        → 讀 figma Power 出版面與 Design Token
   cocos-team        → 讀 kiro-cocos-accelerator 組場景與邏輯
   qa-lead           → functional-tester 驗流程
   compliance-release→ 讀 kiro-game-compliance-expert（若要上架）
   ```

**你在這個流程裡只需要回答一次「可以」。** 這是諮詢模式的設計目的。

---

### 情境 B：你已經有規格

> **你**：幫我在 Unity 實作這個技能樹，規格在 `docs/skill-tree-spec.md`。

**會發生什麼**：

1. Producer **不會**進入諮詢模式（你已給出規格，`advisory-mode.md` 明訂此時不要反覆確認已決定的事）

2. 直接建立 Task Contract → 委派 `tech-lead` → 轉 `unity-team`

3. `unity-team` 讀 `kiro-unity-accelerator` 的對應 steering（場景組裝／腳本／Build），**不憑印象猜 MCP 工具名稱**

4. 完成後寫 Delivery Manifest 到 `.kiro/state/handoffs/TASK-xxx.delivery.yaml`

5. `tech-lead` 做 code review → Producer 回報你

**若規格有數值問題**（例如技能點成長曲線不合理），`unity-team` 不會自己改——它回報給 Producer，Producer 轉 `rpg-systems-expert`（讀 `kiro-rpg-systems-expert`）判斷。

---

### 情境 C：純諮詢，不要動工

> **你**：如果我要做一個有 PvP 的卡牌手遊，最大的技術風險是什麼？

**會發生什麼**：

Producer 判斷這是分析型問題 → 委派多個 Lead 平行評估 → 彙整成一份風險清單，**不建立任何 Task Contract、不產出任何檔案**。

- `tech-lead`：PvP 同步架構（會拉 `mmo-expert` 判斷是 T1 還是 T2 級別，見 `kiro-mmo-netcode-expert` 的 scope 分級）
- `domain-lead` → `card-game-expert`：power creep 是長期結構性風險
- `design-lead`：先手優勢是卡牌 PvP 的結構問題，必須量化測量
- `qa-lead`：對戰模擬的樣本量需求（±1pp 精度需約 9,604 局）

**要動工才會動工。** 分析型問題不會意外產出一堆檔案。

---

## 4. 資訊流動的檔案地圖

Agent 之間**沒有即時對話**（subagent 彼此隔離），全靠讀寫共享檔案 + Producer 轉述。所以想知道現況，看這些檔案：

| 你想知道 | 看哪裡 |
|---------|-------|
| 遊戲設計現在長什麼樣 | `.kiro/steering/project/gdd.md` |
| 美術與聲音方向定了什麼 | `.kiro/steering/project/style-guide.md` |
| 現在有哪些任務、狀態如何 | `.kiro/state/tasks.yaml` |
| 某個任務交付了什麼、有什麼已知問題 | `.kiro/state/handoffs/<contract_id>.delivery.yaml` |
| 實際產出的資產檔 | `shared/`（models／textures／sprites／audio／locales／sim…） |
| 現在算哪個階段、能不能往下推 | `.kiro/steering/project/milestones.md` |
| 哪個 agent 有哪個 Power | `.kiro/steering/global/powers-registry.md` |

**交付紀錄是 append-only**：要更正就補一則新的，不改舊的。這樣才追溯得回去。

---

## 5. Power 在哪一層被讀取（最容易誤解的一點）

```
producer          ← 沒有 Power。它的知識是「怎麼調度」，寫在自己的 prompt
   │
5 個 Lead          ← 沒有 Power。它們的知識是「跨 Specialist 的取捨」，也在 prompt
   │
Specialist        ← ✅ Power 在這一層被讀取
   │
   └─ 29 個有 Power／19 個沒有
```

**為什麼 Lead 沒有 Power**：Lead 的價值來自「知道 A 方案和 B 方案的取捨」，那是本專案的組織知識，不屬於任何單一領域。給 Lead 掛 Power 會讓它偏向那個領域。

**為什麼不是每個 agent 都有 Power**：`producer`、5 個 Lead、`game-designer`、`level-designer`、`narrative-designer`、`combat-designer`、`functional-tester`、`performance-tester`、`marketing-team`、`ui-programmer` 這些角色的工作是協調與整合，領域知識分散在多個 Power 裡，不該複製一份給它們。

**4 個還在等 Power**：`systems-programmer`、`localization-team`、`devops-team`、`usability-tester`。它們目前用自己 prompt 裡的知識運作——**能用，但那些知識沒有經過驗證，也不會自動更新**。

---

## 6. 卡住時怎麼救

| 症狀 | 原因 | 怎麼救 |
|------|------|-------|
| Agent 說找不到 Power steering | 該 Power 沒安裝 | Kiro → Powers 面板安裝 `hoycdanny/<power名>`；或確認 `~/.kiro/powers/installed/` 有沒有它 |
| Agent 開始問一堆技術問題 | 沒進入諮詢模式 | 明說「我不懂這部分，請直接給我建議和預設值」 |
| Agent 用了不存在的 MCP 工具 | 沒讀 Power 就動手（Steering-First 沒被遵守） | 要求它「先讀對應 Power 的 steering 再操作」。這是已知弱點，見 §7 |
| 兩個 Specialist 給的數值互相矛盾 | 缺 Lead 整合 | 回到 Producer，要求委派對應 Lead 做整合審查 |
| 產出跑到奇怪的地方 | 沒讀 `asset-standards.md` | 指出正確落地目錄（`shared/<類型>/`）與命名規範 |
| Beta 之後有人想加新功能 | 沒走 Change Request | 要求 Producer 產出 CR（`contracts.md`），你核准後才執行 |
| Agent 說「應該沒問題」但沒給證據 | 驗證紀律沒被遵守 | 要求它給出可查核的數字（Power 的 `verification-policy.md` 都有規定該附什麼） |

---

## 7. 已知限制（誠實聲明）

這些是**架構層面的已知弱點**，不是 bug。知道它們才不會被意外結果嚇到。

### 7.1 Steering-First 沒有機制強制

Power 內的 `hooks/pre-*-tool.json` 本來設計成「呼叫工具前強迫先讀 steering」，但**subagent 執行環境不會觸發 Hooks**（Kiro 官方文件明載）。本專案的 Pipeline 全走 subagent 委派，所以那道防護完全不生效。

**後果**：Specialist 可能跳過 steering 直接動手。**這是 `unity-team` 當初累積 7 處假 API 的同一個成因。**

**你能做的**：看到 agent 直接開始操作工具而沒有讀 steering 的跡象時，明確要求它先讀。

### 7.2 兩層委派（Producer → Lead → Specialist）未經完整驗證

Kiro 官方文件對多層巢狀 subagent 委派沒有明確保證。本專案採用兩層模型，但**若某次巢狀委派失敗，退化策略是 Producer 直接委派該 Specialist**（見 `producer.md` 的分派規則表）。

**你會看到的症狀**：Lead 回報「無法委派」或直接自己回答了 Specialist 該回答的事。這時要求 Producer 改用直接委派。

### 7.3 subagent 拿不到 Specs、不觸發 Hooks

`.kiro/specs/` 的內容在 subagent 內讀不到。**所以不要把關鍵規格只放在 Specs 裡**——要放進 `gdd.md` 或明確寫進委派 prompt。

### 7.4 Power 的知識邊界

25 個 Power 裡的內容分三級（`HIGH`／`MEDIUM`／`UNVERIFIED`）。**`UNVERIFIED` 佔比不小**，尤其：

- 所有「業界平均」數字（留存率、ARPPU、常見 TTK 區間、coyote time 毫秒數）
- 所有法規細節（會改版）
- 所有引擎端匯入行為（Power 沒有連線可驗證）
- 所有平台延遲數字

Agent 應該照實轉述這個等級。**看到具體數字但沒標等級時，問它「這個數字是可推導的還是需要校準的」。**

### 7.5 沒有人能替你判斷「好不好玩」

這是所有 Power 都寫進能力邊界的一條。數值可以模擬到不會崩、關卡可以驗證到走得通、效能可以量到達標，但**手感與趣味只有真人試玩能判斷**。

`usability-tester` 提供評估框架，**但它不能真的玩遊戲**。這條限制沒有繞道。

---

## 8. 建議的第一步

如果這是全新專案，最有價值的起手不是把 48 個 agent 都跑一遍，而是：

**做一款極小但端到端跑通的遊戲，直到有可執行的 build。**

理由：這條 Pipeline 有很多接縫（Contract 傳遞、資產落地、Delivery Manifest、引擎匯入、Build 產物），**每個接縫都可能在真實使用時才暴露問題**。用一個小到兩天能做完的東西驗證整條路，比先把設計文件寫得很完整有價值得多。

驗證清單：

- [ ] Producer 能正確偵測類型與引擎，並委派到對的 Lead
- [ ] Lead 能轉發給 Specialist 並收回結果（驗證 §7.2 的兩層委派）
- [ ] Specialist 真的讀了 Power steering（問它引用了哪一份）
- [ ] 資產落在 `shared/` 的正確目錄且命名合規
- [ ] Delivery Manifest 有被寫出來，下游讀得到
- [ ] 引擎 Team 能匯入上游資產並產出可執行 build
- [ ] QA 能回報出至少一個帶 severity 的問題（驗證 `bug-severity.md` 有被遵守）

跑完這一輪，你會知道哪些接縫是真的通的、哪些只是文件上寫著。
