# Closing Kit Checklist（結案資料包）

> 這是 [Kiro Multi-Agent Game Studio](../README.md) 的深入文件之一。完整索引見 README 的「深入文件（Reference）」。
>
> 參考《The Game Production Handbook》Ch18 的結案資料包概念，但降規格對應本專案的 Solo Dev 現況：書中情境假設要交接給另一個真人團隊維護，本專案目前是個人開發者自用，所以這份清單的實際價值主要是**版本封存**（例如出貨/上架前，或未來真的要交接/找人協作時）。不是每次小版本更新都要跑一次，建議在 Gold milestone（見 `.kiro/steering/project/milestones.md`）或重大版本封存時使用。

## 使用時機

- 準備上架/出貨前，確認所有交付物都有歸檔且可追溯
- 未來若要把專案交接給其他人（外包、新加入的協作者）
- 想封存某個版本的完整狀態（例如「v1.0 出貨版」），方便日後回溯

## 檢查清單

### 程式碼

- [ ] 目標引擎專案（Unity/Godot/Unreal/Cocos）的完整專案檔已在版本控制內，且能從乾淨的 clone 直接開啟
- [ ] `.kiro/` 內所有 Agent 定義、steering 檔都在版本控制內
- [ ] 沒有未提交（uncommitted）的重大變更遺留在工作目錄
- [ ] 已知的技術債/未解決問題整理成清單（可用 `.kiro/state/tasks.yaml` 或 issue tracker 的 open 項目）

### 資產

- [ ] `shared/` 內所有中介資產（模型/貼圖/音訊/locale）都已透過 Git LFS 追蹤（見 `.gitattributes`）
- [ ] 沒有「僅存在於某人電腦本機」的關鍵資產——所有東西都進了版本控制
- [ ] 資產命名符合 `asset-standards.md` 規範，方便日後查找

### 文件

- [ ] `.kiro/steering/project/gdd.md` 內容反映目前實際的遊戲設計（不是過時版本）
- [ ] `.kiro/steering/project/style-guide.md` 反映目前實際採用的美術/聲音風格
- [ ] `.kiro/steering/project/milestones.md` 標注目前實際到達的階段
- [ ] 若有走過 Change Request（見 `contracts.md`），重大變更已記入 `gdd.md`「變更紀錄」
- [ ] 若有做過 Postmortem（見 `gdd.md`「Postmortem」章節），已補上

### 工具與環境

- [ ] `.kiro/settings/mcp.json` 使用的 MCP Server 清單與版本已記錄（方便日後重建環境）
- [ ] 任何需要的環境變數/API Key 的**名稱**（不是值）已列出，說明去哪裡取得
- [ ] `README.md`「快速開始」的安裝步驟仍然可用（建議實際照著跑一次驗證）

### 法遵與上架（若適用）

- [ ] `compliance-release` 產出的分級/隱私/送審清單狀態已更新到最新
- [ ] 老虎機類專案：認證/牌照相關文件狀態已確認（見 `slot-game-expert` 與 `compliance-release` 的交接）

## 誠實聲明

- 本清單目前**沒有自動化檢查機制**——沒有工具能自動掃描並勾選以上項目，需要使用者或 `producer` 人工過一輪確認。
- 這份清單參考書中框架但大幅簡化，因為本專案目前規模（Solo Dev）不需要書中完整的多團隊交接流程；若未來擴充到 Small Team / Studio 規模（見 `docs/architecture-and-process.md`「漸進式擴展指南」），可能需要補充更完整的交接文件。
