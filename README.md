# Kiro Multi-Agent Game Studio

透過多 Agent 架構模擬完整遊戲開發團隊，各 Agent 各司其職，透過 MCP (Model Context Protocol) 串接外部工具（ComfyUI、Blender、Figma、Unity），實現 AI 驅動的遊戲開發 Pipeline。

## 核心概念

- **多層級 Agent 架構**：Portfolio Orchestrator → Creative Director / Producer → Team Leads → Specialist Agents
- **MCP 工具整合**：ComfyUI（圖像生成）、Blender（3D 建模）、Figma（UI 設計）、Unity（遊戲引擎）
- **多團隊並行（V-Team）**：命名空間隔離、資源配額、Resource Broker 算力排隊
- **品質保證**：Review Gate 機制、自動化測試、衝突升級協議

## 文件結構

```
docs/
├── ARCHITECTURE.md          # 完整架構設計文件
├── TEAM_STRUCTURE.md        # 團隊角色與職責
├── PIPELINE.md              # 資產 Pipeline 與工作流程
├── GOVERNANCE.md            # 治理機制（衝突升級、成本控管、鎖定）
└── MULTI_TEAM.md            # 多 V-Team 隔離與資源分配
```

## 工具鏈

| 工具 | 用途 | 整合方式 |
|------|------|----------|
| **ComfyUI** | 2D 圖像生成（概念圖、貼圖、Sprite、UI 素材） | MCP Server |
| **Blender** | 3D 建模、動畫、渲染 | MCP Server |
| **Figma** | UI/UX 設計、規格匯出 | MCP Server |
| **Unity** | 遊戲引擎（資產匯入、場景組裝、程式撰寫、測試打包） | MCP Server + Editor Script + CLI |
| **Git** | 版本控制（程式碼 + 資產） | MCP Server / CLI |
| **Project Tracker** | 任務追蹤、Sprint 看板 | MCP Server |

## 架構總覽

```
┌─────────────────────────────────────────────────┐
│          Portfolio Orchestrator                   │
│    （管理多個 V-Team、跨團隊資源仲裁）            │
└────────────────────┬────────────────────────────┘
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
   ┌─────────┐ ┌─────────┐ ┌─────────┐
   │ V-Team 1│ │ V-Team 2│ │ V-Team N│
   └────┬────┘ └────┬────┘ └────┬────┘
        │            │            │
        ▼            ▼            ▼
┌──────────────────────────────────────────┐
│  Creative Director + Producer             │
│  （願景守護 + 執行管理）                   │
├──────────────────────────────────────────┤
│  Design Lead │ Art Lead │ Tech Lead │ ...│
├──────────────────────────────────────────┤
│  Specialist Agents（實際執行工作）         │
├──────────────────────────────────────────┤
│  Resource Broker（算力閘門）              │
└──────────────────────────────────────────┘
```

## 遊戲生命週期

```
Concept → Prototype → Vertical Slice → Alpha → Beta → Gold → Live
```

## 自動化等級

| 等級 | 描述 | 適用場景 |
|------|------|----------|
| Level 0 | Agent 建議 → 人工執行 | 平台審核提交 |
| Level 1 | Agent 執行 → 人工 Review | 3D 建模、程式碼、數值平衡 |
| Level 2 | Agent 執行 → 自動 Review → 人看例外 | 概念圖生成、Build |
| Level 3 | 全自動 | Unit Test、Icon 批量生成 |

## 待確認事項

- [ ] 遊戲類型（2D / 3D / 混合）
- [ ] 目標平台（PC / Mobile / Console / WebGL）
- [ ] 音訊需求（是否整合 AI 音樂生成）
- [ ] 多人連線需求
- [ ] Monetization 模型（買斷 / F2P+IAP / 訂閱）
- [ ] 團隊規模偏好
- [ ] 是否需要 LiveOps / 持續營運

## License

MIT
