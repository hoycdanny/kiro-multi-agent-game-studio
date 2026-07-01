# 資產 Pipeline 與工作流程

## 遊戲生命週期與里程碑框架

### 全局里程碑（Milestones）

```
Concept → Prototype → Vertical Slice → Alpha → Beta → Gold → Live
```

| 里程碑 | 目標 | 產出 | 通過條件 |
|--------|------|------|----------|
| **Concept** | 確認遊戲方向 | GDD 初稿、核心玩法描述、市場定位 | Creative Director 簽核 |
| **Prototype** | 驗證核心玩法是否「好玩」 | 可玩原型（允許醜、允許 bug） | 核心循環有趣，值得繼續投入 |
| **Vertical Slice** | 展示最終品質的一小段完整體驗 | 1 個完整關卡/場景，所有團隊產出整合 | 品質代表最終遊戲水準 |
| **Alpha** | 所有核心功能完成 | 可從頭玩到尾（內容可不完整） | 所有系統可運作，無 blocker |
| **Beta** | 所有內容完成，進入除錯 | 完整遊戲內容 | 無新功能，只修 bug |
| **Gold** | 可出貨版本 | 通過平台審核的 Build | 無 Critical bug，效能達標 |
| **Live** | 上線營運 | 持續更新版本 | 監控指標正常 |

### 各階段的 Agent 參與度

```yaml
concept:
  active: [creative_director, game_designer, narrative_designer]
  minimal: [art_lead]  # 只做風格探索
  inactive: [programmer, qa, audio]

prototype:
  active: [game_designer, gameplay_programmer]
  minimal: [art_lead]  # placeholder art OK
  inactive: [marketing, compliance, audio]
  key_principle: "速度優先，品質不重要"

vertical_slice:
  active: [ALL]  # 所有團隊都參與
  key_principle: "品質代表最終水準"

alpha:
  active: [ALL]
  key_principle: "功能完整性優先"

beta:
  active: [qa_lead, programmer, art_lead]  # 修 bug 為主
  minimal: [design]  # 不再加新設計
  key_principle: "穩定性優先，凍結功能"

gold:
  active: [qa_lead, devops_agent, compliance_agent]
  key_principle: "通過審核，準備出貨"

live:
  active: [producer, devops_agent, marketing_agent, balance_tester]
  key_principle: "數據驅動迭代"
```

---

## 功能開發流程（單一功能顆粒度）

```
Phase 0: Prototype / 驗證
────────────────────────
目的：用最低成本驗證「這個功能好不好玩/有沒有用」
  → Gameplay Programmer 快速搭建原型（placeholder art）
  → Usability Tester 或人類玩家試玩
  → Game Designer 根據回饋決定：繼續 / 修改方向 / 砍掉
  → [Review Gate: Concept Validation]
  ※ 只有通過驗證的功能才進入正式製作流程

Phase 1: Design（設計）
────────────────────────
Producer 接收已驗證的需求
  → Game Designer 產出系統規格
  → UX Designer 產出操作流程 Wireframe
  → Narrative Designer 產出相關劇情/對話
  → [Review Gate: Design Review]

Phase 2: Pre-production（前期製作）── 平行展開
────────────────────────
Art Director 分派美術任務：
  ├─ Concept Artist (ComfyUI) → 概念圖
  ├─ UI Artist (Figma) → UI Layout
  └─ VFX Artist → 特效規格

Tech Lead 拆分程式任務：
  ├─ Gameplay Programmer → 核心邏輯
  ├─ UI Programmer → UI 實作
  └─ Systems Programmer → 底層系統

Audio Director：
  └─ Sound Designer → 音效製作

Phase 3: Production（製作）── 依賴驅動
────────────────────────
Texture Artist (ComfyUI) → PBR 貼圖
3D Modeler (Blender) → 模型 + 套貼圖
Animator (Blender) → 骨骼 + 動畫
Technical Artist → Shader 優化、效能驗證
  → [Review Gate: Art Review]

Gameplay Programmer → 遊戲邏輯 C#
  → Unit Test 自動執行
  → [Review Gate: Code Review]

Phase 4: Integration（整合）
────────────────────────
Unity Import Agent：
  - 匯入美術資產（自動設定 Import Settings）
  - 生成 Prefab
  - 組裝場景
  - 連接程式邏輯與美術資產
  - 設定 Lighting / NavMesh / Camera

Phase 5: QA & Polish（測試與打磨）
────────────────────────
QA Agent：
  - 功能測試
  - 數值平衡測試
  - 效能測試
  - 可用性測試（Usability）
  → Bug 回報到對應團隊
  → 修復循環（max_iterations 限制）
  → [Review Gate: Release Review]

Phase 6: Build & Deploy
────────────────────────
DevOps Agent + Unity CLI Batch Mode：
  - 打包目標平台
  - 產出 Build Report
  - 觸發 CI/CD Pipeline

Phase 7: Platform Certification（平台審核）
────────────────────────
Compliance Agent：
  - 平台技術需求檢查（TRC/XR/Lot Check）
  - 分級內容審查
  - 隱私政策/GDPR 合規
  → 提交平台審核
  → 處理審核回饋（reject → 修正 → 重新提交）

Phase 8: Post-launch / LiveOps（上線後）
────────────────────────
持續循環：
  - 數據收集（留存率、付費率、卡關點）
  - Balance Tester 根據數據調整數值
  - Marketing Agent 產出更新公告/社群內容
  - 新功能回到 Phase 0 重新驗證
```

---

## 資產 Pipeline 細節

### ComfyUI Workflow Templates

```yaml
comfyui_workflows:
  - name: "character_concept"
    template: "workflows/character_multi_angle.json"
    params: [prompt, style, pose, background]
    output: 角色概念圖（正面、側面、背面）

  - name: "pbr_texture"
    template: "workflows/pbr_generation.json"
    params: [material_type, color_palette, tiling]
    output: Albedo + Normal + Roughness + AO

  - name: "sprite_sheet"
    template: "workflows/sprite_animation.json"
    params: [character_prompt, action, frame_count]
    output: Sprite Sheet PNG

  - name: "style_transfer"
    template: "workflows/img2img_style.json"
    params: [input_image, target_style, strength]
    output: 風格轉換後圖片

  - name: "ui_icon_batch"
    template: "workflows/icon_generation.json"
    params: [icon_descriptions, style, size]
    output: 一批 UI Icon
```

### Blender Pipeline

```yaml
blender_pipeline:
  modeling:
    input: 概念圖 + Asset Spec
    process: 建模 → UV 展開 → 材質套用
    output: .fbx / .glb

  animation:
    input: 3D 模型 + 動作需求
    process: 骨骼綁定 → 動畫製作 → 匯出
    output: .fbx (with animation clips)

  rendering:
    input: 場景 / 模型
    process: 設定燈光 → 渲染
    output: 預覽圖 / Marketing 素材
```

### Unity Integration

```yaml
unity_integration:
  asset_import:
    method: "File-based (AssetPostprocessor)"
    auto_settings:
      model: { scale: 0.01, generate_collider: true, rig_type: "auto" }
      texture: { max_size: "platform_dependent", compression: "auto" }
      audio: { load_type: "streaming_for_bgm, decompress_for_sfx" }

  scene_assembly:
    method: "Unity MCP Server (Editor Plugin)"
    capabilities:
      - 擺放 GameObject
      - 設定 Component
      - Lighting / NavMesh / Camera
      - Trigger / Event 配置

  code_generation:
    method: "File-based (直接寫 .cs)"
    standards:
      namespace: "GameForge.{Module}"
      naming: "PascalCase public, _camelCase private"
      pattern: "Composition over Inheritance, ScriptableObject data-driven"
      documentation: "XML Doc for all public members"

  build_test:
    method: "CLI Batch Mode"
    commands:
      test: "Unity -batchmode -runTests -testResults results.xml"
      build: "Unity -batchmode -executeMethod BuildScript.Build"
```

---

## Agent 間通訊協定

### Asset Contract（資產契約）

```yaml
asset_request:
  id: "vt_001.weapon_sword_01"
  team_id: "vt_001"
  type: "3d_model"          # 3d_model | texture | sprite | audio | prefab
  spec:
    poly_budget: 5000
    texture_size: 1024
    style: "stylized_fantasy"
    reference_images: ["ref_sword_01.png"]
  unity_import:
    scale: 0.01
    generate_collider: true
    prefab_path: "Assets/Prefabs/Weapons/"
  metadata:
    priority: "high"
    assigned_to: "modeler_3d"
    depends_on: ["concept_art_sword_01"]
    deadline: "sprint_3"
  cost_budget:
    max_llm_tokens: 50000
    max_comfyui_generations: 10
    max_blender_operations: 20
```

### Task Contract（任務契約）

```yaml
task:
  id: "TASK-042"
  title: "實作戰鬥傷害計算"
  assigned_to: "gameplay_programmer"
  input:
    - design_spec: "docs/combat_system_spec.yaml"
    - dependencies: ["health_system", "buff_system"]
  output:
    - code: "Assets/Scripts/Combat/DamageCalculator.cs"
    - tests: "Assets/Tests/Combat/DamageCalculatorTests.cs"
  acceptance_criteria:
    - "傷害公式符合 design_spec 定義"
    - "所有 Unit Test 通過"
    - "處理 edge case（0 防禦、無敵狀態）"
  review_gate: "code_review"
  cost_budget:
    max_llm_tokens: 100000
    priority_weight: 0.8
```
