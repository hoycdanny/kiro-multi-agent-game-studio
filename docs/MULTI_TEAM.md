# 多 V-Team 隔離與資源分配機制

當同時運作多個遊戲專案（v-Team）時，需要解決三個核心問題：
1. 團隊間不能互相污染資產
2. 共享工具不能被同時搶用
3. 資源緊張時要有仲裁規則

---

## 一、命名空間隔離（Namespace）

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

---

## 二、共享資源競爭：Resource Broker Agent

ComfyUI/Blender/Unity MCP Server 是有限的運算資源（尤其 GPU），多個 v-Team 不能同時搶。

```yaml
resource_broker_agent:
  role: "Resource Broker"
  manages:
    - comfyui_mcp_pool      # 可能有多個 ComfyUI 實例組成的池
    - blender_mcp_pool
    - unity_mcp_pool
  responsibilities:
    - 接收各 v-Team 的工具使用請求
    - 依優先級與配額排隊分配
    - 侵佔檢測（是否有 team 超用配額）
    - 釋放資源通知下一個排隊者

  queue_policy:
    algorithm: "priority_weighted_fifo"
    # A 級團隊請求優先處理，同級內先進先出
    starvation_guard: true
    # 防止低優先級團隊永遠排不到，超過 N 次跳過後強制插隊一次
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

---

## 三、資源配額（Quota）

每個 v-Team 依 `priority_tier` 拿到不同額度：

```yaml
quota_vt_001:
  tier: "A"
  daily_limits:
    comfyui_generations: 500
    llm_tokens: 5_000_000
    blender_render_hours: 10
  overflow_policy: "borrow_from_pool"
  # 額度用完後可向「共享池」借用，但優先級降為最低

quota_vt_002:
  tier: "B"
  daily_limits:
    comfyui_generations: 200
    llm_tokens: 2_000_000
    blender_render_hours: 4
  overflow_policy: "block_until_reset"
  # 用完就得等下個週期，不能借
```

---

## 四、跨團隊協調層：Portfolio Orchestrator

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

---

## 五、Steering 文件三層結構

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

---

## 六、並發鎖定機制

同 team 內也可能有多個 agent 同時想改同一資產：

```yaml
asset_lock:
  asset_id: "vt_001.weapon_sword_01"
  locked_by: "animator_agent"
  locked_at: "2026-07-01T10:30:00+08:00"
  lock_type: "exclusive"   # exclusive | shared_read
  auto_release_after_sec: 1800
  # 超時自動釋放，避免 agent crash 後鎖死資源
```

---

## 七、跨團隊資產借用規則

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
