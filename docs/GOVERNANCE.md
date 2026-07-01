# 治理機制

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

## 衝突升級機制（Conflict Escalation）

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

---

## 資產鎖定機制（Asset Locking）

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
    asset_id: "vt_001.weapon_sword_01"
    locked_by: "animator_agent"
    locked_at: "2026-07-01T10:30:00+08:00"
    lock_type: "exclusive"   # exclusive | shared_read
    auto_release_after_sec: 1800
```

---

## Token / 成本預算控管

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
Level 3: 全自動（僅限低風險任務）
```

各環節建議等級：

| 環節 | 建議等級 | 理由 |
|------|----------|------|
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
