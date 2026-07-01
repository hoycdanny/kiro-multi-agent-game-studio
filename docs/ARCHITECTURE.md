# Game Forge Agents - 多 Agent 遊戲開發架構規劃

## 概述

透過多 Agent 架構模擬完整遊戲開發團隊，各 Agent 各司其職，透過 MCP (Model Context Protocol) 串接外部工具（ComfyUI、Blender、Figma、Unity），實現 AI 驅動的遊戲開發 Pipeline。

---

## 工具鏈

| 工具 | 用途 | 整合方式 |
|------|------|----------|
| **ComfyUI** | 2D 圖像生成（概念圖、貼圖、Sprite、UI 素材） | MCP Server |
| **Blender** | 3D 建模、動畫、渲染 | MCP Server |
| **Figma** | UI/UX 設計、規格匯出 | MCP Server |
| **Unity** | 遊戲引擎（資產匯入、場景組裝、程式撰寫、測試打包） | MCP Server + Editor Script + CLI Batch Mode |
| **Git** | 版本控制（程式碼 + 資產） | MCP Server / CLI |
| **Project Tracker** | 任務追蹤、Sprint 看板、進度總覽 | MCP Server (Linear/Jira/自建) |
| **Localization Tool** | 多語系管理 | MCP Server (Crowdin/Lokalise/自建) |

---

## 團隊架構

### Layer 0：Strategic（戰略層）

```yaml
creative_director:
  role: "Creative Director"
  responsibilities:
    - 守護遊戲願景與核心體驗
    - 定義遊戲的「感覺」和創意方向
    - 跨團隊創意決策的最終仲裁者
    - 確保所有產出符合統一的藝術/設計方向
  decision_authority:
    - 玩法方向爭議的最終決定權
    - 美術風格方向的最終決定權
    - 「這個東西該不該進遊戲」的否決權
  relationship:
    - 與 Producer 平行：Creative Director 管「做什麼」，Producer 管「怎麼做」
```

### Layer 1：Orchestration（指揮層）

```yaml
producer_agent:
  role: "Producer / Orchestrator"
  responsibilities:
    - 接收需求，拆解為可執行任務
    - 決定任務優先級和依賴關係
    - 分派任務到對應團隊 Agent
    - 管理衝突升級機制
    - 追蹤進度，確認交付物品質
    - 里程碑規劃與進度匯報
    - Token/成本預算控管
  decision_authority:
    - 需求變更的影響評估
    - 資源分配（哪個 Agent 先做）
    - 品質門檻（是否通過 Review Gate）
    - 排程與 deadline 管理
  tools: [project_tracker_mcp]
```

### Layer 2：Team Leads（團隊主管層）

```yaml
design_lead:
  role: "Design Lead"
  sub_agents: [game_designer, level_designer, narrative_designer, ux_designer, economy_designer, combat_designer]
  input: 創意方向（from Creative Director）+ 執行需求（from Producer）
  output: GDD, 系統規格, 關卡設計文件, 對話腳本, 經濟模型

art_lead:
  role: "Art Director"
  sub_agents: [concept_artist, modeler_3d, animator, ui_artist, vfx_artist, texture_artist, technical_artist]
  input: GDD + 參考圖 + 風格指南
  output: 美術資產（概念圖、模型、貼圖、動畫、UI、特效）

tech_lead:
  role: "Tech Lead"
  sub_agents: [gameplay_programmer, systems_programmer, ui_programmer, devops_agent]
  input: 設計規格 + 美術資產
  output: 可執行遊戲版本

audio_lead:
  role: "Audio Director"
  sub_agents: [sound_designer, composer]
  input: GDD + 場景氛圍描述
  output: 音效、配樂

qa_lead:
  role: "QA Lead"
  sub_agents: [functional_tester, balance_tester, performance_tester, usability_tester]
  input: 遊戲版本 + 設計文件
  output: Bug 報告、測試結果、改善建議

marketing_agent:
  role: "Marketing / Community"
  input: 遊戲版本 + 美術資產 + GDD
  output: 商店頁面文案、宣傳素材、社群溝通稿、Trailer 腳本

compliance_agent:
  role: "Legal / Compliance"
  input: 遊戲內容 + 素材來源紀錄
  output: 分級審查報告、版權風險評估、平台合規檢查
```

### Layer 3：Specialist Agents（專業執行層）

#### Design Team

```yaml
game_designer:
  tools: [document_editor]
  produces:
    - 遊戲設計文件 (GDD)
    - 系統規格（戰鬥、成長系統等）
    - 數值平衡表
    - 公式定義

economy_designer:
  tools: [document_editor, simulation_tools]
  produces:
    - 經濟模型（貨幣流入/流出）
    - 商城定價策略
    - IAP 設計
    - 營收平衡模擬
  note: "與 game_designer 的數值平衡分開，避免遊戲性與營收邏輯互相干擾"

combat_designer:
  tools: [document_editor, simulation_tools]
  produces:
    - 戰鬥系統規格
    - 技能設計（數值、效果、冷卻）
    - 敵人 AI 行為模式
    - 戰鬥節奏設計
  note: "適用於動作/RPG 類型，若非戰鬥導向遊戲可併入 game_designer"

level_designer:
  tools: [unity_mcp, tilemap_editor]
  produces:
    - 關卡佈局
    - 觸發器配置
    - 路徑規劃
    - 難度曲線

narrative_designer:
  tools: [document_editor, dialogue_tree_tool]
  produces:
    - 世界觀設定
    - 劇情大綱 / 支線任務
    - 對話樹（Yarn / Ink 格式）
    - 角色背景故事

ux_designer:
  tools: [figma_mcp]
  produces:
    - 操作流程設計
    - Wireframe
    - 回饋機制規格（打擊感、UI 動態）
    - 新手引導流程
```

#### Art Team

```yaml
concept_artist:
  tools: [comfyui_mcp]
  produces:
    - 角色概念圖（多角度）
    - 場景氛圍圖
    - 道具設計圖
    - 風格探索變體

texture_artist:
  tools: [comfyui_mcp]
  produces:
    - PBR Texture（Albedo, Normal, Roughness, AO）
    - Sprite Sheet
    - UI Icons / Elements
    - Seamless Tileable Textures

modeler_3d:
  tools: [blender_mcp]
  input_from: [concept_artist, texture_artist]
  produces:
    - 3D 模型 + UV 展開
    - 材質套用
    - LOD 版本
    - Collider Mesh

animator:
  tools: [blender_mcp]
  input_from: [modeler_3d]
  produces:
    - 骨骼綁定 (Rigging)
    - 動畫片段 (.anim)
    - Shape Keys（表情）
    - Animation Controller 設定

ui_artist:
  tools: [figma_mcp, comfyui_mcp]
  produces:
    - UI Layout（Figma）
    - UI 素材生成（ComfyUI）
    - Design Token（顏色、字型、間距）
    - 互動狀態定義（Normal, Hover, Pressed, Disabled）

vfx_artist:
  tools: [unity_mcp, comfyui_mcp]
  produces:
    - 粒子特效
    - Shader / Shader Graph
    - 序列幀動畫
    - Post-processing 設定

technical_artist:
  role: "Technical Artist（橋接 Art 與 Tech）"
  tools: [blender_mcp, unity_mcp, code_editor]
  produces:
    - Shader 優化（確保美術效果在效能預算內）
    - Rig 效能調整
    - 貼圖壓縮策略
    - Art Pipeline 工具開發
    - LOD 策略制定
  note: "解決美術想要的效果 vs 程式端效能限制之間的落差"
```

#### Programming Team

```yaml
gameplay_programmer:
  tools: [code_editor, unity_mcp]
  produces:
    - 遊戲邏輯（MonoBehaviour / ECS）
    - 狀態機（戰鬥、AI、動畫）
    - 技能系統
    - 互動系統

systems_programmer:
  tools: [code_editor]
  produces:
    - 存檔 / 讀檔系統
    - 資源管理（Addressables）
    - 網路層（若有多人）
    - 事件系統 / 訊息匯流排
    - Object Pooling

ui_programmer:
  tools: [code_editor, unity_mcp]
  produces:
    - UI 綁定（UI Toolkit / UGUI）
    - 動態 UI 邏輯
    - Localization 支援
    - UI 動畫 / 轉場

devops_agent:
  role: "DevOps / Infrastructure"
  tools: [git_mcp, unity_cli, ci_cd_tools]
  produces:
    - CI/CD Pipeline 設定與維護
    - 自動化 Build 腳本
    - 版本控制策略執行（分支管理、LFS 設定）
    - 部署流程（測試版/正式版）
    - 開發環境自動化設定
```

#### Audio Team

```yaml
sound_designer:
  tools: [audio_generation_mcp]
  produces:
    - 音效（攻擊、環境、UI）
    - Audio Event 設定
    - 空間音效配置

composer:
  tools: [audio_generation_mcp]
  produces:
    - 背景音樂
    - 戰鬥音樂
    - 動態音樂系統規格
```

#### QA Team

```yaml
functional_tester:
  tools: [unity_test_runner]
  produces:
    - Unit Test（EditMode / PlayMode）
    - Integration Test
    - Bug 報告

balance_tester:
  tools: [simulation_tools]
  produces:
    - 數值模擬結果
    - 平衡性報告
    - 經濟系統壓力測試

performance_tester:
  tools: [unity_profiler, unity_cli]
  produces:
    - FPS / Memory / Draw Call 報告
    - 效能瓶頸分析
    - 平台相容性報告

usability_tester:
  role: "Usability / Playtest Agent"
  tools: [analytics_tools, session_recorder]
  produces:
    - 新手引導有效性評估
    - 操作直覺性評估
    - 卡關點分析
    - 玩家行為預測模擬
  note: "部分情境需要真人玩家參與（Human-in-the-loop），不能完全自動化"
```

---

## 遊戲生命週期與里程碑框架

### 全局里程碑（Milestones）

```
┌─────────────────────────────────────────────────────────────────────┐
│  Concept → Prototype → Vertical Slice → Alpha → Beta → Gold → Live │
└─────────────────────────────────────────────────────────────────────┘
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

### 功能開發流程（單一功能顆粒度）

```
Phase 0: Prototype / 驗證（新增）
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

Phase 7: Platform Certification（平台審核）── 新增
────────────────────────
Compliance Agent：
  - 平台技術需求檢查（TRC/XR/Lot Check）
  - 分級內容審查
  - 隱私政策/GDPR 合規
  → 提交平台審核
  → 處理審核回饋（reject → 修正 → 重新提交）

Phase 8: Post-launch / LiveOps（上線後）── 新增
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
  id: "weapon_sword_01"
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
    priority_weight: 0.8  # 高優先級任務可以用更多資源
```

---

## Review Gate（品質關卡）

```yaml
review_gates:
  concept_validation:
    reviewer: creative_director
    criteria:
      - 是否符合遊戲願景
      - 核心循環是否有趣（原型驗證）
      - 值得投入正式製作資源

  design_review:
    reviewer: design_lead
    criteria:
      - 是否符合核心遊戲體驗目標
      - 系統間是否有矛盾
      - 數值是否合理（可模擬驗證）

  art_review:
    reviewer: art_lead + technical_artist
    criteria:
      - 風格一致性（與 Style Guide 對照）
      - 技術規格（面數、貼圖大小、命名規範）
      - 效能可行性（Technical Artist 確認）
      - 視覺品質

  code_review:
    reviewer: tech_lead
    criteria:
      - 命名規範
      - 效能考量
      - 測試覆蓋率
      - 架構合理性

  release_review:
    reviewer: producer
    criteria:
      - 所有功能符合 Design Spec
      - 無 Critical / Major Bug
      - 效能達標（目標 FPS、Memory）
      - 所有 Review Gate 已通過

  compliance_review:
    reviewer: compliance_agent
    criteria:
      - 內容分級合規（ESRB / PEGI / CERO）
      - 平台技術需求滿足
      - AI 生成素材的版權風險已評估
      - 隱私政策 / GDPR 合規
```

---

## 治理機制

### 衝突升級機制（Conflict Escalation）

```yaml
conflict_resolution:
  levels:
    - level: 1
      description: "同團隊內的分歧"
      resolver: "Team Lead"
      timeout: "1 iteration"
      example: "兩個 concept 方案選哪個"

    - level: 2
      description: "跨團隊衝突（Art vs Tech）"
      resolver: "Producer + 相關 Team Leads"
      timeout: "2 iterations"
      example: "美術想要的粒子特效超出效能預算"

    - level: 3
      description: "影響遊戲方向的決策"
      resolver: "Creative Director"
      timeout: "1 iteration（優先處理）"
      example: "砍掉一個系統 vs 延期"

  common_conflicts:
    art_vs_tech:
      scenario: "美術效果超出效能預算"
      resolution_path:
        1. Technical Artist 評估是否有優化方案
        2. 若無法優化 → Art Lead 與 Tech Lead 協商妥協方案
        3. 若無法達成共識 → Producer 根據優先級裁決
        4. 若涉及遊戲願景 → Creative Director 最終決定

    design_vs_schedule:
      scenario: "設計想加功能但時間不夠"
      resolution_path:
        1. Producer 評估影響範圍
        2. 提出選項：砍 scope / 延期 / 加資源
        3. Creative Director 確認是否影響核心體驗
        4. Producer 做最終排程決定

  rules:
    - "任何 Agent 可以提出異議，但必須附上理由和替代方案"
    - "逾時未解決自動升級到下一層"
    - "Creative Director 的創意方向決策為最終決定"
    - "Producer 的排程/資源決策為最終決定"
```

### 資產鎖定機制（Asset Locking）

```yaml
asset_locking:
  strategy: "Pessimistic Lock（悲觀鎖）"
  rules:
    - "修改資產前必須先取得鎖定"
    - "鎖定期間其他 Agent 只能讀取，不能修改"
    - "鎖定超時自動釋放（防止死鎖）"
    - "衝突時由 Producer 裁定優先級"
  
  implementation:
    lock_registry: "shared_context/asset_locks.yaml"
    max_lock_duration: "1 sprint"
    conflict_resolution: "higher priority task wins"
    
  example:
    - agent: "texture_artist"
      locks: "assets/characters/hero/texture_albedo.png"
      reason: "重新生成角色貼圖"
      expires: "2024-01-15T18:00:00"
```

### Token / 成本預算控管

```yaml
cost_management:
  budget_levels:
    per_task:
      description: "每個 Task Contract 自帶預算上限"
      fields: [max_llm_tokens, max_comfyui_generations, max_blender_operations]
      
    per_sprint:
      description: "每個 Sprint 的總預算"
      allocated_by: producer
      breakdown:
        design: 15%
        art_generation: 35%  # ComfyUI 最耗資源
        programming: 25%
        qa: 15%
        other: 10%
    
    per_milestone:
      description: "每個里程碑的預算上限"
      review: "Producer 在每個 milestone 結束時檢討用量"

  controls:
    - "低優先級任務不得超過 budget 的 50%"
    - "超出預算需要 Producer 核准追加"
    - "每日產出 cost report，異常自動告警"
    - "Level 3 自動化任務設定嚴格 cost cap"
    
  tracking:
    tool: project_tracker
    metrics: [tokens_used, generations_count, api_calls, compute_time]
```

---

## 共享知識庫（Shared Context）

所有 Agent 共享以下資料，確保一致性：

| 文件 | 用途 | 維護者 |
|------|------|--------|
| GDD (Game Design Document) | 遊戲設計的單一真相來源 | Game Designer |
| Style Guide | 美術風格指南 | Art Director |
| Technical Spec | 技術規範（平台、效能預算） | Tech Lead |
| Asset Registry | 已有資產清單 + 鎖定狀態 | Producer / DevOps |
| World Bible | 世界觀、角色設定 | Narrative Designer |
| Code Architecture Doc | 程式架構、模組關係圖 | Tech Lead |
| Economy Model | 經濟系統模型、營收目標 | Economy Designer |
| Compliance Checklist | 各平台合規需求清單 | Compliance Agent |
| Cost Dashboard | 即時成本追蹤 | Producer |

---

## 自動化等級

```
Level 0: Agent 產出建議 → 人工執行
Level 1: Agent 執行 → 人工 Review（建議起始等級）
Level 2: Agent 執行 → 自動 Review → 人只看例外
Level 3: 全自動（僅限低風險任務，如 Icon 批量生成）
```

各環節建議等級：
| 環節 | 建議自動化等級 | 理由 |
|------|---------------|------|
| 概念圖生成 | Level 2 | 快速迭代，人看最終選擇即可 |
| 3D 建模 | Level 1 | 品質要求高，需人工確認 |
| 程式碼撰寫 | Level 1 | 需要 Code Review |
| Unit Test | Level 3 | 可全自動 |
| 數值平衡 | Level 1 | 影響遊戲體驗，需人確認 |
| UI Icon 批量生成 | Level 3 | 低風險，可全自動 |
| Build & Deploy | Level 2 | CI/CD 自動化，人看結果 |
| 平台審核提交 | Level 0 | 高風險，必須人工確認 |

---

## 版本控制策略

```yaml
version_control:
  tool: "Git"
  large_files: "Git LFS"
  lfs_tracked:
    - "*.fbx"
    - "*.glb"
    - "*.png"
    - "*.psd"
    - "*.wav"
    - "*.mp3"
    - "*.unitypackage"
  
  branching:
    strategy: "Git Flow variant"
    branches:
      main: "穩定的可出貨版本"
      develop: "開發整合分支"
      feature/*: "單一功能開發"
      art/*: "美術資產開發"
      hotfix/*: "緊急修復"
    
  commit_convention:
    format: "[team][type] description"
    examples:
      - "[art][add] hero character model v1"
      - "[prog][fix] combat damage calculation edge case"
      - "[design][update] GDD combat system spec"

  mcp_integration:
    - "所有 Agent 透過 Git MCP 進行版本操作"
    - "自動 commit message 生成"
    - "PR 自動建立並指派 reviewer"
```

---

## 多 V-Team 隔離與資源分配機制

當同時運作多個遊戲專案（v-Team）時，需要解決三個核心問題：
1. 團隊間不能互相污染資產
2. 共享工具不能被同時搶用
3. 資源緊張時要有仲裁規則

### 一、命名空間隔離（Namespace）

每個 v-Team 有唯一 `team_id`，所有產出物、任務、資產都強制帶前綴。

```yaml
# team_registry.yaml — 全局團隊註冊表
teams:
  - team_id: "vt_001"
    project_name: "Project Aurora"
    status: "active"          # active | paused | archived
    created_at: "2026-06-01"
    priority_tier: "A"        # A(旗艦) / B(常規) / C(實驗性)
    assigned_producer: "producer_agent_001"
    resource_quota_ref: "quota_vt_001"

  - team_id: "vt_002"
    project_name: "Project Ember"
    status: "active"
    priority_tier: "B"
    assigned_producer: "producer_agent_002"
    resource_quota_ref: "quota_vt_002"
```

Asset Contract 強制帶 team_id 前綴：

```yaml
asset_request:
  id: "vt_001.weapon_sword_01"     # 強制前綴，不可省略
  team_id: "vt_001"
  type: "3d_model"
  spec:
    poly_budget: 5000
    ...
```

Asset Registry 按 team 分區存儲，查詢時預設只看自己 team 的資產，除非明確要求跨 team 共享。

### 二、共享資源競爭：Resource Broker Agent

ComfyUI/Blender/Unity MCP Server 是有限的運算資源（尤其 GPU），多個 v-Team 不能同時搶。

```yaml
resource_broker_agent:
  role: "Resource Broker"
  manages:
    - comfyui_mcp_pool
    - blender_mcp_pool
    - unity_mcp_pool
  responsibilities:
    - 接收各 v-Team 的工具使用請求
    - 依優先級與配額排隊分配
    - 侵佔檢測（是否有 team 超用配額）
    - 釋放資源通知下一個排隊者

  queue_policy:
    algorithm: "priority_weighted_fifo"
    starvation_guard: true
    max_skip_before_boost: 3
```

請求/分配流程：

```yaml
resource_request:
  request_id: "req_20260701_0007"
  team_id: "vt_002"
  requested_tool: "comfyui_mcp"
  workflow: "character_concept"
  estimated_duration_sec: 45
  priority_tier: "B"
  status: "queued"        # queued | granted | running | completed | rejected
  queued_at: "2026-07-01T10:22:00+08:00"
```

### 三、資源配額（Quota）

每個 v-Team 依 `priority_tier` 拿到不同額度：

```yaml
quota_vt_001:
  tier: "A"
  daily_limits:
    comfyui_generations: 500
    llm_tokens: 5_000_000
    blender_render_hours: 10
  overflow_policy: "borrow_from_pool"

quota_vt_002:
  tier: "B"
  daily_limits:
    comfyui_generations: 200
    llm_tokens: 2_000_000
    blender_render_hours: 4
  overflow_policy: "block_until_reset"
```

### 四、跨團隊協調層：Portfolio Orchestrator

多團隊情境下需要比 Producer 更高一層的協調者：

```yaml
portfolio_orchestrator:
  role: "Portfolio Orchestrator"
  manages: [producer_agent_001, producer_agent_002, ...]
  responsibilities:
    - 跨團隊資源衝突仲裁（Resource Broker 排不開時上報到這裡）
    - 團隊優先級動態調整（例如某專案臨近上線，臨時升級 tier）
    - 團隊生命週期管理（新建 / 暫停 / 歸檔 v-Team）
    - 跨團隊知識複用授權
  decision_authority:
    - 是否核准新 v-Team 的建立
    - 團隊間的資源再分配
    - 是否允許某 team 借用其他 team 的已完成資產
  escalation_from: [producer_agent]
```

### 五、Steering 文件三層結構

```
.kiro/steering/
├── global/                      # 所有 v-Team 共用，Portfolio Orchestrator 維護
│   ├── company_art_style.md
│   ├── naming_conventions.md
│   └── code_standards.md
│
├── teams/
│   ├── vt_001/                  # 該 team 專屬，team 的 Producer 維護
│   │   ├── gdd.md
│   │   ├── style_guide.md
│   │   └── asset_registry.md
│   └── vt_002/
│       ├── gdd.md
│       └── style_guide.md
│
└── agents/                      # 單個 agent 角色的行為限制
    ├── gameplay_programmer.md
    └── concept_artist.md
```

衝突規則：
- Global 層與 Team 層矛盾時，以 Portfolio Orchestrator 的裁決為準
- Team 不能自行繞過 global steering，需走審批

### 六、並發鎖定機制

```yaml
asset_lock:
  asset_id: "vt_001.weapon_sword_01"
  locked_by: "animator_agent"
  locked_at: "2026-07-01T10:30:00+08:00"
  lock_type: "exclusive"   # exclusive | shared_read
  auto_release_after_sec: 1800
```

### 七、跨團隊資產借用規則

```yaml
cross_team_asset_sharing:
  policy: "conditional_sharing"

  rules:
    # 可直接借用（不需審批）
    tier_1_free:
      conditions:
        - "已通過 Review Gate"
        - "標記為 reusable"
        - "借用方只讀、不修改"
      examples:
        - 通用 Shader
        - 基礎 UI Component
        - 環境音效

    # 需審批（Asset 擁有者 team 的 Lead 同意）
    tier_2_approved:
      conditions:
        - "借用方需要修改/衍生"
        - "或該資產是團隊核心識別物"
      process:
        1. 借用方提出 request
        2. 擁有方 Team Lead 審核
        3. 核准後 fork 一份到借用方 namespace
        4. 修改在 fork 上進行，不影響原版
      examples:
        - 角色模型（改顏色/裝備變體）
        - 特定風格的 Texture

    # 禁止借用
    tier_3_blocked:
      conditions:
        - "未通過 Review Gate"
        - "標記為 team_exclusive"
        - "涉及劇情/品牌核心差異化"
      examples:
        - 主角設計（各團隊的遊戲識別度）
        - 未完成的 WIP 資產
```

借用職責分工：
- **Resource Broker**：只管算力分配，不管資產借用
- **Portfolio Orchestrator**：負責 tier_2 審批流程
- **Asset Registry**：記錄每個資產的 sharing_tier 標記

---

## 待確認事項

- [ ] 遊戲類型（2D / 3D / 混合）
- [ ] 目標平台（PC / Mobile / Console / WebGL）
- [ ] 音訊需求（是否整合 AI 音樂生成，如 Suno/Udio）
- [ ] 多人連線需求（是否需要 Network Agent）
- [ ] Monetization 模型（買斷 / F2P+IAP / 訂閱）
- [ ] 團隊規模偏好（精簡版 vs 完整版 Agent 配置）
- [ ] 目標受眾與分級（影響 Compliance Agent 工作量）
- [ ] 是否需要 LiveOps / 持續營運
