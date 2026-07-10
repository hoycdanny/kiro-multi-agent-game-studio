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
  assigned_to: "engineering/unity-team"   # 依目標引擎填入：unity-team | godot-team | unreal-team | cocos-team
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
  id: "vt_001.weapon_sword_01"
  team_id: "vt_001"
  type: "3d_model"          # 3d_model | texture | sprite | audio | prefab
  spec:
    poly_budget: 5000
    style: "stylized_fantasy"
    reference_images: ["ref_sword_01.png"]
  textures:                 # 由 art/comfyui-team 填入，art/blender-team 讀取
    albedo: null
    normal: null
    roughness: null
  engine_import:             # 依目標引擎調整欄位，交給對應 engineering/{engine}-team 使用
    engine: "Unity"           # Unity | Godot | Unreal | Cocos Creator
    scale: 0.01
    generate_collider: true
  metadata:
    priority: "high"
    assigned_to: "art/blender-team"
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

資料夾（`art/`、`design/`、`engineering/`、`qa/`、`orchestration/`）只是 `.kiro/agents/` 底下的檔案組織，**不是呼叫名稱的一部分**。Kiro 依 frontmatter 的 `name` 註冊 Agent 並在 Agent Selector / slash command / subagent 委派中以該名稱辨識。

目前已註冊的扁平名稱：`producer`、`portfolio-orchestrator`、`game-designer`、`slot-game-expert`、`economy-designer`、`ui-ux-team`、`localization-team`、`comfyui-team`、`blender-team`、`animator`、`audio-team`、`unity-team`、`godot-team`、`unreal-team`、`cocos-team`、`devops-team`、`functional-tester`、`balance-tester`、`compliance-release`。

## 團隊隔離（team_id）

Agent 讀寫團隊專屬檔案時，路徑一律用 `<team_id>` 佔位，實際值由 Producer 於委派時傳入（預設 `vt_001`）：

- 設計 / 風格：`.kiro/steering/teams/<team_id>/gdd.md`、`.kiro/steering/teams/<team_id>/style-guide.md`
- 任務看板：`.kiro/state/teams/<team_id>/tasks.yaml`

**不要把 `vt_001` 寫死**——文件中的 `vt_001` 僅為範例。這樣才能讓多個 V-Team（`vt_001`、`vt_002`…）並行而不互相污染上下文。

## Subagent 委派機制（Kiro 原生，取代舊的手動轉接）

Kiro 原生支援 subagent 委派：主 Agent 用 `Use the "<name>" subagent to …` 語法即可觸發，**不需要特別的 `subagent` 工具權限**，Specialist 執行完會自動把結果回傳給主 Agent。因此 Producer 應**主動自動委派**，不再要求使用者手動切換 Agent Selector 貼上 Contract。

**尚待實測的邊界（誠實聲明）**：
- subagent 執行環境是隔離的獨立 context window，因此**委派時必須把完整 Contract 與所有檔案路徑寫進 Prompt**，否則 Specialist 會缺上下文。
- subagent 內**不會觸發 Hooks、也拿不到 Specs**（見 Kiro 官方 Subagents 文件）。
- **多層巢狀委派**（`portfolio-orchestrator` → `producer` → Specialist，共三層）尚未在本專案完整驗證。若巢狀委派失敗，退化策略是由 `producer` 作為主 Agent 逐一委派各 Specialist，不強求三層自動串接。
