---
inclusion: always
---

# Milestone 驗收標準（Milestones）

> **這份檔案是什麼**：定義本專案從 Prototype 到 Gold 各階段的**驗收標準（Exit Criteria）**——「這個階段算不算做完」的判斷依據，補上 `gdd.md`（設計内容）與本檔（階段驗收）之間的落差。`producer` 在判斷 Pipeline 是否可以往下一階段推進、`qa-lead` 在判斷 go/no-go 時都應該先讀這份檔案。
>
> **維護者**：`producer`（跨階段推進的判斷者），驗收標準的內容由使用者與 `producer` 討論後填入。目前為骨架，各階段的具體標準需依實際專案填寫，**不要自行假設已達標**。

## 階段總覽

對應 `docs/architecture-and-process.md` 的「功能開發生命週期」：

```
Concept ──── Prototype ──── Vertical Slice ──── Alpha ─── Beta ─── Gold
```

## Prototype（原型驗證）

- **目的**：用最低成本驗證核心玩法是否值得投入
- **Exit Criteria**（尚未定義，建議項目）：
  - [ ] 核心循環（Core Loop）可操作一次完整流程（即使是 placeholder art）
  - [ ] 使用者確認「這個玩法值得繼續做」
- **誰簽核**：使用者本人（`creative-director` 若已啟用可協助判斷是否符合願景）

## Vertical Slice（垂直切片）

- **目的**：用最終品質（美術/音效/手感）完整呈現一小段可代表整款遊戲的內容
- **Exit Criteria**（尚未定義，建議項目）：
  - [ ] 至少一個系統/關卡是「最終美術+音效+完整邏輯」，不是 placeholder
  - [ ] 核心玩法的手感/數值已經過至少一輪 `balance-tester` 或實測調整
- **誰簽核**：使用者本人

## Alpha（內容完整、允許有 bug）

- **目的**：所有規劃內容都已存在（可能還有 bug），可以開始完整地玩過一輪
- **Exit Criteria**（尚未定義，建議項目）：
  - [ ] `gdd.md` 中規劃的系統都已至少有一版實作
  - [ ] 無 S1/Critical 等級的已知問題阻擋主要流程（見 `.kiro/steering/global/bug-severity.md`）
  - [ ] 美術/音效資產覆蓋率達到可評估整體風格一致性的程度
- **誰簽核**：使用者本人，`qa-lead` 提供品質現況報告

## Beta（功能凍結、修 bug 為主）

- **目的**：不再新增功能，專注修 bug 與調數值
- **Exit Criteria**（尚未定義，建議項目）：
  - [ ] 功能凍結（Feature Freeze）：不再接受新功能請求，新需求走 Change Request（見 `contracts.md`）並延到下一版
  - [ ] 無未解決的 S1/S2 等級問題（見 `bug-severity.md`）
  - [ ] `balance-tester` 已完成至少一輪數值驗證（若專案含老虎機/F2P 經濟）
- **誰簽核**：使用者本人，`qa-lead` 給 go/no-go 建議

## Gold（可出貨版本）

- **目的**：正式出貨/上架的版本
- **Exit Criteria**（尚未定義，建議項目）：
  - [ ] 無任何未解決的 S1 等級問題
  - [ ] `compliance-release` 確認分級/送審/合規清單已齊備（若適用）
  - [ ] Build 已通過 `devops-team` 的產物驗證（若已啟用）
- **誰簽核**：使用者本人

## 使用規則

1. 每個階段的 Exit Criteria 目前是**建議骨架**，不是本專案已驗證的固定標準——使用者應依實際遊戲類型與規模調整勾選項，`producer` 不要自行認定「已達標」。
2. 判斷是否可以進入下一階段時，`producer` 先讀本檔對應階段的 Exit Criteria，逐項確認狀態，缺項要明確列出，而不是籠統回報「差不多完成了」。
3. 階段推進的決定權在使用者，`producer` / `qa-lead` 只負責提供現況與是否達標的判斷依據。
4. 每次階段推進後，在下方「變更紀錄」記一筆。

## 變更紀錄

| 日期 | 變更 | 負責人 |
|------|------|--------|
| - | 建立骨架 | producer |
