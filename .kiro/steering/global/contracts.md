---
inclusion: always
---

# Agent 間通訊協定（Contract）

Agent 之間不靠隨意對話交付需求，而是透過以下兩種標準化 Contract。

## Task Contract（程式 / 設計任務用）

```yaml
task:
  id: "TASK-042"
  title: "任務標題"
  assigned_to: "unity-team"                # 依目標引擎填入：unity-team | godot-team | unreal-team | cocos-team
  engine: "Unity"                          # Unity | Godot | Unreal | Cocos Creator
  input:
    - design_spec: "路徑或描述"
    - dependencies: ["依賴的其他任務或系統"]
  output:
    - code: "預期產出路徑"
    - tests: "測試檔案路徑（如適用）"
  acceptance_criteria:
    - "驗收條件 1"
    - "驗收條件 2"
  review_gate: "code_review | design_review"
```

## Asset Contract（美術 / 音效資產用）

```yaml
asset_request:
  id: "weapon_sword_01"
  type: "3d_model"          # 3d_model | texture | sprite | audio | prefab
  spec:
    poly_budget: 5000
    style: "stylized_fantasy"
    reference_images: ["ref_sword_01.png"]
  textures:                 # 由 comfyui-team 填入，blender-team 讀取
    albedo: null
    normal: null
    roughness: null
  engine_import:             # 依目標引擎調整欄位，交給對應引擎 team 使用
    engine: "Unity"           # Unity | Godot | Unreal | Cocos Creator
    scale: 0.01
    generate_collider: true
  metadata:
    priority: "high"
    assigned_to: "blender-team"
    depends_on: []
```

## 核心 Pipeline 流動方式

```
User → Producer（建立 Contract，偵測引擎與遊戲類型）
      → design/game-designer（規格，若需要）
      → design/slot-game-expert（老虎機數學模型/RNG/合規，若偵測到該類型）
      → design/ui-ux-team（Figma UI/UX 版面 + Design Token + 切圖規格，若含介面需求）
      → art/comfyui-team（貼圖 / UI 切圖素材，若需要）
      → art/blender-team（建模 + 套貼圖，2D 遊戲可跳過）
      → engineering/{unity,godot,unreal,cocos}-team（依偵測到的引擎分派，場景組裝 + 遊戲邏輯 + Build）
      → Producer（確認交付 → Git commit）
```

每一步的 Contract 由 Producer 負責串接：把前一個 Team 的交付物（例如貼圖路徑、.fbx 路徑）填進下一個 Team 的 Contract 裡再轉交。最後一步永遠分派給**使用者指定的引擎對應的 Team**，而不是固定分派給 `unity-team`。

## Agent 委派命名規範（所有 Agent 必讀）

委派 / 呼叫其他 Agent 時，**一律使用該 Agent frontmatter 的扁平 `name`**，不要加資料夾前綴：

- ✅ 正確：`Use the "blender-team" subagent to …`、`Use the "unity-team" subagent to …`
- ❌ 錯誤：`Use the "art/blender-team" agent …`、`Use the "engineering/unity-team" agent …`

所有 Agent 檔案已平鋪存放在 `.kiro/agents/` 根目錄下（例如 `orchestration_producer.md`、`design_game-designer.md`）。檔名前綴（如 `orchestration_`、`design_` 等）僅作為組織與區分用途，**不是呼叫名稱的一部分**。Kiro 依 frontmatter 的 `name` 註冊 Agent 並在 Agent Selector / slash command / subagent 委派中以該名稱辨識。

目前已註冊的扁平名稱：`creative-director`、`producer`、`game-designer`、`design-lead`、`slot-game-expert`、`fish-game-expert`、`shooter-expert`、`mmo-expert`、`rpg-systems-expert`、`card-game-expert`、`puzzle-match3-expert`、`platformer-expert`、`roguelike-expert`、`strategy-expert`、`simulation-expert`、`rhythm-expert`、`narrative-adventure-expert`、`economy-designer`、`ui-ux-team`、`localization-team`、`art-lead`、`comfyui-team`、`blender-team`、`animator`、`audio-team`、`technical-artist`、`tech-lead`、`unity-team`、`godot-team`、`unreal-team`、`cocos-team`、`devops-team`、`qa-lead`、`functional-tester`、`balance-tester`、`performance-tester`、`compliance-release`。

## Subagent 委派機制（Kiro 原生，取代舊的手動轉接）

Kiro 原生支援 subagent 委派：主 Agent 用 `Use the "<name>" subagent to …` 語法即可觸發。**注意：主 Agent 必須在其 YAML frontmatter 的 `tools` 列表中包含 `"subagent"` 工具權限**，Specialist 執行完會自動把結果回傳給主 Agent。因此 Producer 應**主動自動委派**，不再要求使用者手動切換 Agent Selector 貼上 Contract。

**已知邊界（誠實聲明）**：
- subagent 執行環境是隔離的獨立 context window，因此**委派時必須把完整 Contract 與所有檔案路徑寫進 Prompt**，否則 Specialist 會缺上下文。
- subagent 內**不會觸發 Hooks、也拿不到 Specs**（見 Kiro 官方 Subagents 文件）。
- **多層巢狀委派不支援**：要委派的 agent 必須自身在 `tools` 含 `subagent`，各 Specialist 都沒有此權限，因此只支援單層「Producer → Specialist」；由 `producer` 逐一委派各 Specialist（Producer 已具備 `subagent` 權限）。

## 檔案共享與交接（精簡協作規範）

所有 agent 都有 `read` 權限，可讀 repo 內任何檔案——**agent 之間的「溝通」就是透過讀寫這些共享檔案**（subagent 彼此隔離、沒有即時對話，一律走檔案 + Producer 轉述）。

**共享位置（大家都讀得到）**
- 設計真相：`.kiro/steering/project/`（gdd.md、style-guide.md）
- 全域規範：`.kiro/steering/global/`（本檔、asset-standards.md）
- 任務與交接：`.kiro/state/`（tasks.yaml、handoffs/）
- 實際產出：`shared/`（Agent 檔案共享中轉站；命名避開 `assets` 以免和引擎內部 `Assets/`、`db://assets/` 混淆，見 shared/README.md 的落地目錄）

**規則**
1. **動工前先讀**：對應的 Contract ＋ 上游的 Delivery Manifest（`handoffs/`）＋ gdd/style-guide ＋ 相關產出路徑。不要在沒讀上游交付的情況下開始。
2. **交付後寫一則 Delivery Manifest** 到 `.kiro/state/handoffs/<contract_id>.delivery.yaml`，讓下游（含各引擎 team）讀得到你產出了什麼、在哪、有什麼已知問題。
3. **blocker / 提問**：一句話記在該任務的 tasks.yaml 條目或 Delivery Manifest 的 `notes`，由 Producer 讀取並轉述給相關 agent；跨團隊的重要決策補進 gdd.md「變更紀錄」。
4. **append-only**：交付紀錄只增不改，要更正就補一則新的（可追溯）。

**Delivery Manifest 格式（交付回執，補完 Contract 的回程）**

```yaml
delivery:
  contract_id: "TASK-042"
  by: "blender-team"
  outputs: ["shared/models/character_hero_01.fbx"]
  acceptance:
    - { criteria: "poly ≤ 8000", status: "pass" }
  known_issues: ["roughness 貼圖尚缺"]
  next: "交 unity-team 匯入，import scale 0.01"
  notes: ""
```

Producer 是中樞：負責把上游的 Delivery Manifest 內容填進下一個 agent 的委派 prompt，確保跨 team（含 Unity/Godot/Unreal/Cocos）都讀得到彼此的產出與資料。
