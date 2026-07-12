# 參考資料

> 這是 [Kiro Multi-Agent Game Studio](../README.md) 的深入文件之一。完整索引見 README 的「深入文件（Reference）」。

## 成本估算


### 單一 Indie 遊戲（從 Concept 到 Gold，約 26 週，願景估算）

| 階段 | LLM Tokens | ComfyUI 次數 | 預估成本 |
|------|-----------|-------------|----------|
| Concept (2w) | 2M | 50 | $30-50 |
| Prototype (4w) | 5M | 100 | $80-120 |
| Vertical Slice (6w) | 10M | 300 | $200-400 |
| Alpha (8w) | 15M | 500 | $300-600 |
| Beta (4w) | 5M | 50 | $80-150 |
| Gold (2w) | 2M | 10 | $30-50 |
| **合計** | **~39M** | **~1010** | **$720-1370** |

> 使用本地 LLM + 本地 ComfyUI（SDXL）可降至 $100-300（僅電費）。**本專案目前尚未產出任何實際資產，以上為原始估算，未經本專案實測驗證。**

### 省錢策略

- 本地 LLM 跑 Level 3 任務（Unit Test、Icon 批量）→ 省 50-70%
- ComfyUI 本地 SDXL（需 12GB VRAM）→ 圖像生成成本趨近 0
- 只在 Code Review / Design Review 用高級模型 → 省 30-40%
- Prototype 階段嚴格篩選，早期砍掉不好玩的設計

---

## 錯誤處理與退化策略


### MCP 故障

| 工具掛了 | Retry（願景） | Fallback（願景） | 本專案現況 |
|---------|-------|----------|-----------|
| ComfyUI | 3 次（exponential backoff） | 通知用戶手動操作 WebUI | `comfyui-team.md` 目前做法更簡單：最多重試 2 次，連續失敗就停止並回報具體錯誤，不會自動退化成操作 WebUI |
| Blender | 2 次 | 匯出 Python Script，用戶手動執行 | `blender-team.md` 目前做法更簡單：連線失敗直接回報並停止，不會自動重試或匯出腳本 |
| Unity MCP | 1 次 | 產出 .cs，用戶在 Editor 操作 | `unity-team.md` 目前做法：連線失敗（`project_info` 讀不到）直接回報並停止；操作逾時（Unity 忙碌中）重試 1 次，不會自動退化成只產出 .cs |
| Godot MCP | 1 次 | 產出 .gd，用戶在 Editor 操作 | `godot-team.md` 目前做法：連線失敗（`get_project_info` 失敗）直接回報並停止 |
| Unreal MCP | 1 次 | 產出說明文件，用戶手動操作 | `unreal-team.md` 目前做法：連線失敗直接回報並停止；已知會 crash 的 `ce` command 絕不作為 fallback 使用 |
| Cocos MCP | 1 次 | 產出 .ts，用戶在 Editor 操作 | `cocos-team.md` 目前做法：連線失敗（fetch failed）直接回報並停止 |
| GitHub Projects | 2 次 | 記錄到本地 tasks.yaml | `github` MCP 已寫進 `mcp.json`；在下載 `github-mcp-server` binary + 填 PAT 實際連上前，以本地 `tasks.yaml` 為 fallback |

### 品質不達標

```
max_iterations: 3

概念圖被退：調 prompt → 加 negative → 換 seed/模型 → 3 次後升級 Art Lead 人工介入
程式被退：根據意見修改 → 重跑 Test → 3 次後標記 needs_human_review
```

> `blender-team.md` 和 `functional-tester.md` 都已寫入「max_iterations: 3」的限制，超過會停止並回報使用者，而非無限重試。

### 成本超支

```
80% → 警告 + 切換便宜模型
100% → 暫停 → Producer 決定：追加預算 / 降低品質要求 / 人工接手
```

> 本專案目前無自動化成本追蹤，此策略僅為文字提醒，實際判斷依賴使用者自行留意 token 用量。

---

## 設計依據


本框架的團隊分工參考了遊戲產業通用的六大學科分類（Design、Art、Engineering、Audio、QA、Production），並結合 Agile/Scrum 的迭代開發方法。AI Agent 特有的機制（token budget、MCP 整合、Resource Broker 等）為原創設計，本專案的「誠實聲明」慣例（明確標註哪些是願景、哪些是實際可用）則是在實作過程中因應 Kiro Custom Agent 的實際限制而額外加入的原則，不屬於下列參考文獻範疇。

### 參考文獻

| # | 文獻 | 作者 | 出版 | ISBN |
|---|------|------|------|------|
| 1 | *The Game Production Handbook*, 3rd Edition | Heather Maxwell Chandler | Jones & Bartlett Learning, 2014 | 978-1-4496-8809-7 |
| 2 | *Agile Game Development: Build, Play, Repeat*, 2nd Edition | Clinton Keith | Addison-Wesley (Pearson), 2020 | 978-0-1365-2781-7 |
| 3 | IGDA Curriculum Framework (2008) | IGDA Education SIG | IGDA | — |

### 連結

- Chandler：[O'Reilly](https://www.oreilly.com/library/view/the-game-production/9781449688097/) ｜ [AbeBooks](https://www.abebooks.com/9781449688097/Game-Production-Handbook-Chandler-Heather-1449688098/plp)
- Keith：[Pearson](https://www.pearson.com/store/p/agile-game-development-build-play-repeat/P100002783425/9780136527817) ｜ [O'Reilly](https://www.oreilly.com/library/view/agile-game-development/9780136204831)
- IGDA Curriculum Framework：[Google Drive（IGDA 官方）](https://drive.google.com/file/d/1s9cMaSIjeD2ERhjfCMsh9f1-qs-GJx_A/view) ｜ [IGDA Education SIG](https://igda.org/sigs/game-education/)
- IGDA Game Industry Standards：[igda.org](https://igda.org/resources/game-industry-standards/)
- Blender MCP：[官方頁面](https://www.blender.org/lab/mcp-server/#llm-client) ｜ [原始碼](https://projects.blender.org/lab/blender_mcp)
- Model Context Protocol：[modelcontextprotocol.io](https://modelcontextprotocol.io/)

---

## 共享知識庫


所有 Agent 透過 `.kiro/steering/` 共享以下資料：

| 文件 | 用途 | 維護者（願景） | 本專案現況 |
|------|------|--------|-----------|
| `.kiro/steering/global/asset-standards.md` | 命名規範、3D/音訊/動畫技術規範、資產落地目錄（`shared/`） | art-lead | 已建立，內容完整 |
| `.kiro/steering/global/contracts.md` | Task/Asset Contract 格式 + Change Request（防 Feature Creep） + 委派命名規範 + subagent 機制 | producer | 已建立，內容完整 |
| `.kiro/steering/global/bug-severity.md` | Bug 嚴重度分級（S1-S4）+ release 門檻，QA 全線共用標準 | qa-lead | 已建立，內容完整 |
| `shared/README.md` + `.gitattributes` | 交付物落地目錄結構 + Git LFS 規則 | producer / devops-team | 已建立 |
| `.kiro/steering/project/gdd.md` | 遊戲設計的單一真相來源（GDD）+ Postmortem 範本 | game-designer | 已建立骨架，章節內容待填寫 |
| `.kiro/steering/project/style-guide.md` | 美術風格指南 | art-lead | 已建立骨架，章節內容待填寫 |
| `.kiro/steering/project/milestones.md` | Prototype→Gold 各階段驗收標準（Exit Criteria） | producer | 已建立骨架，各階段標準待依專案填寫 |
| Technical Spec | 技術規範（平台、效能預算） | tech-lead | 尚未建立 |
| Asset Registry | 已有資產清單 + 鎖定狀態 | producer | 尚未建立（目前尚無任何實際資產產出） |
| World Bible | 世界觀、角色設定 | narrative-designer | 尚未建立 |
| Code Architecture | 程式架構、模組關係圖 | tech-lead | 尚未建立 |
| Cost Dashboard | 即時成本追蹤 | producer | 尚未建立（無自動化追蹤機制） |
| `.kiro/state/tasks.yaml` | GitHub Projects 接上前的本地任務記錄 fallback | producer | 已建立，目前為空清單 |

---

## 專案檔案結構（本專案實際結構）


```
kiro-multi-agent-game-studio/
├── .kiro/
│   ├── agents/                                 # 46 個 Agent（委派用扁平 name，資料夾僅為組織）
│   │   ├── orchestration/
│   │   │   ├── creative-director.md            # Layer 0：願景守門 / pillars / 創意仲裁
│   │   │   └── producer.md                     # 唯一調度中樞：拆任務、偵測引擎/遊戲類型、串接 Pipeline、Git commit
│   │   ├── design/
│   │   │   ├── design-lead.md                  # Layer 2：核心設計整合 GDD + design-review gate
│   │   │   ├── domain-lead.md                  # Layer 2：13 類遊戲類型專家的專業審查與轉發
│   │   │   ├── game-designer.md                # GDD、系統規格、數值平衡
│   │   │   ├── level-designer.md               # 關卡佈局/觸發器/難度曲線
│   │   │   ├── narrative-designer.md           # 世界觀/角色/劇情內容/World Bible
│   │   │   ├── combat-designer.md              # 通用戰鬥系統/技能/敵人 AI
│   │   │   ├── ui-ux-team.md                   # UI/UX 版面、Design Token、切圖規格（透過 figma MCP）
│   │   │   ├── economy-designer.md             # F2P 數值、IAP、貨幣、獎勵曲線、變現模型
│   │   │   ├── localization-team.md            # 多語系字串、locale 檔、i18n 落地規格
│   │   │   ├── slot-game-expert.md             # 老虎機：數學模型/RNG/認證合規
│   │   │   ├── fish-game-expert.md             # 魚機/捕魚：命中機率/賠付/RTP/合規
│   │   │   ├── shooter-expert.md               # 射擊：武器/彈道/命中/AI
│   │   │   ├── mmo-expert.md                   # 多人/MMORPG：netcode/伺服器權威
│   │   │   ├── rpg-systems-expert.md           # RPG：屬性/技能/掉落/公式
│   │   │   ├── card-game-expert.md             # 卡牌：數值/combo/平衡
│   │   │   ├── puzzle-match3-expert.md         # 三消/解謎：board 可解性/難度曲線/步數經濟
│   │   │   ├── platformer-expert.md            # 平台/metroidvania：跳躍手感/關卡節奏/gating
│   │   │   ├── roguelike-expert.md             # roguelike/lite：程序生成/build synergy/meta
│   │   │   ├── strategy-expert.md              # 策略 RTS/4X/塔防：兵種相剋/經濟/AI/波次
│   │   │   ├── simulation-expert.md            # 模擬經營/生存/沙盒：生產鏈/供需/自動化
│   │   │   ├── rhythm-expert.md                # 音樂節奏：譜面/判定窗/延遲校正
│   │   │   └── narrative-adventure-expert.md   # 敘事/視覺小說/冒險：分支敘事/旗標/對話樹
│   │   ├── art/
│   │   │   ├── art-lead.md                     # Layer 2：維護 style-guide + 美術/聲音一致性 review
│   │   │   ├── comfyui-team.md                 # 影像：貼圖/sprite/UI 切圖（透過 comfyui）
│   │   │   ├── blender-team.md                 # 靜態 3D 建模 + 套貼圖（透過 blender-mcp）
│   │   │   ├── animator.md                     # rig/綁定/動畫 clip（透過 blender-mcp）
│   │   │   ├── audio-team.md                   # SFX/BGM/voice（透過 comfyui generate_audio）
│   │   │   ├── vfx-artist.md                   # 特效素材/序列幀（透過 comfyui）
│   │   │   └── technical-artist.md             # shader/材質/LOD/優化/匯入管線
│   │   ├── engineering/
│   │   │   ├── tech-lead.md                    # Layer 2：技術架構 + 效能預算 + code-review gate
│   │   │   ├── unity-team.md                   # 場景組裝、遊戲邏輯、Build（透過 unity-mcp）
│   │   │   ├── godot-team.md                   # 場景組裝、GDScript、Export（透過 godot-mcp）
│   │   │   ├── unreal-team.md                  # 關卡組裝、Blueprint、材質（透過 unreal-engine local MCP）
│   │   │   ├── cocos-team.md                   # 場景組裝、TypeScript 元件、Prefab（透過 cocos-creator MCP）
│   │   │   ├── systems-programmer.md           # 引擎無關的存檔/資源管理/事件系統設計
│   │   │   ├── ui-programmer.md                # 把 ui-ux-team 版面綁定成可互動引擎 UI
│   │   │   └── devops-team.md                  # headless build、CI pipeline、產物驗證
│   │   ├── qa/
│   │   │   ├── qa-lead.md                      # Layer 2：測試策略 + 協調四 tester
│   │   │   ├── functional-tester.md            # 功能測試
│   │   │   ├── balance-tester.md               # RTP/經濟數值 Monte Carlo 模擬
│   │   │   ├── performance-tester.md           # FPS/記憶體/draw call profiling
│   │   │   └── usability-tester.md             # 新手引導評估/卡關點分析
│   │   └── publishing/
│   │       ├── compliance-release.md           # 分級、隱私合規、商店素材、送審/認證流程
│   │       └── marketing-team.md               # 商店文案/預告片腳本/新聞稿/社群貼文草稿
│   ├── steering/
│   │   ├── global/
│   │   │   ├── asset-standards.md              # inclusion: always（命名+落地目錄+音訊/動畫規範）
│   │   │   ├── contracts.md                    # inclusion: always（Contract+Change Request+委派命名+subagent）
│   │   │   └── bug-severity.md                 # inclusion: always（Bug 分級 S1-S4 + release 門檻）
│   │   └── project/
│   │       ├── gdd.md                          # ⚠️ 骨架（inclusion: always，遊戲設計單一真相 + Postmortem 範本）
│   │       ├── style-guide.md                  # ⚠️ 骨架（inclusion: always，美術風格）
│   │       └── milestones.md                   # ⚠️ 骨架（inclusion: always，Prototype→Gold 驗收標準）
│   ├── state/
│   │   ├── tasks.yaml                          # GitHub Projects fallback，目前為空
│   │   └── handoffs/                           # Delivery Manifest 交付回執落點
│   └── settings/
│       └── mcp.json                            # blender-mcp / comfyui / unity-mcp / godot-mcp / unreal-engine / cocos-creator / figma
├── shared/                                     # 各 Team 交付物落地目錄／Agent 檔案共享中轉站（見 shared/README.md）
│   ├── concept/ textures/ sprites/ ui/         #    ComfyUI Team
│   ├── models/                                 #    Blender Team
│   ├── rigs/ animations/                       #    Animator
│   ├── audio/{sfx,music,voice}/                #    Audio Team
│   ├── locales/                                #    Localization Team
│   └── sim/                                    #    Balance Tester（RTP/經濟模擬報告）
├── .gitattributes                              # Git LFS 規則（模型/貼圖/音訊/字型/產物）
├── .gitignore                                  # 排除祕密/OS/工具產物，保留 .kiro 與 mcp.json
├── blender/
│   └── README.md                               # 已合併進本檔案「Blender MCP 整合詳解」章節
└── README.md                                   # 本文件
```

願景中尚未擴充的部分主要是 `workflows/`（ComfyUI Workflow Templates）；`combat-designer`、`vfx-artist`、`usability-tester` 已建立，`Audio Lead` 刻意不獨立建立（已併入 `art-lead`）。`docs/` 下另有 `audio-pipeline.md`（配音/音樂 Pipeline，AI vs 真人路徑）與 `closing-kit-checklist.md`（結案資料包檢查清單），未列在上方樹狀圖中（該圖僅列 `.kiro/` 與根目錄結構）。

---
