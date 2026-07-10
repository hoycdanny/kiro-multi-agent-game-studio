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

## 現階段的限制（誠實聲明）

本專案目前**尚未驗證** Kiro 的 Custom Agent 之間是否能透過 subagent 機制自動互相呼叫。
在驗證之前，Producer 的「分派」動作實際上是：

1. 產生上述格式的 Contract
2. 明確告知使用者「請切換到 XX Agent 並貼上這份 Contract」
3. 使用者手動切換 Agent Selector 完成交接

一旦確認 Kiro 支援自動跨 Agent 委派，再回來更新 `orchestration/producer.md` 移除手動交接步驟。
