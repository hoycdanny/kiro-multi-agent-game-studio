# 團隊角色與職責

## 架構層級

```
Layer 0: Strategic（戰略層）── Creative Director
Layer 1: Orchestration（指揮層）── Producer
Layer 2: Team Leads（團隊主管層）── Design/Art/Tech/Audio/QA Lead + Marketing + Compliance
Layer 3: Specialist Agents（專業執行層）── 各專業角色
```

---

## Layer 0：Strategic（戰略層）

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

---

## Layer 1：Orchestration（指揮層）

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

---

## Layer 2：Team Leads（團隊主管層）

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

---

## Layer 3：Specialist Agents（專業執行層）

### Design Team

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

### Art Team

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

### Programming Team

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

### Audio Team

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

### QA Team

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
