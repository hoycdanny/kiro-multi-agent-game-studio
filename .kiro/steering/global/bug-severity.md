---
inclusion: always
---

# Bug 分級標準（Global）

> **這份檔案是什麼**：QA 全線（`functional-tester` / `balance-tester` / `performance-tester` / `usability-tester`）與 `qa-lead` 共用的 Bug 嚴重度分級標準，確保四條測試線回報的問題可以用同一套標準彙整、比較，`qa-lead` 才能做出一致的 go/no-go 判斷。任何 Agent 回報問題時，一律標注下列等級之一，不要自創形容詞（例如「還好」「有點嚴重」）。
>
> **維護者**：`qa-lead`。

## 分級定義

| 等級 | 名稱 | 定義 | 範例 |
|------|------|------|------|
| **S1 / Critical** | 當機級 | 導致當機、存檔損毀、資安漏洞、核心遊戲流程完全無法進行；無 workaround | 遊戲啟動後閃退、老虎機 RTP 計算錯誤導致派彩異常、玩家資料外洩風險 |
| **S2 / Major** | 重大 | 核心功能損壞但有 workaround、明顯影響多數玩家的體驗、重大視覺或音效缺失 | 某關卡卡住需重啟才能過、UI 按鈕在特定解析度下按不到、Big Win 音效完全沒播 |
| **S3 / Minor** | 次要 | 局部功能異常、僅在少見情境觸發、有明顯 workaround、不影響核心體驗 | 特定裝置上圖示模糊、極端數值輸入才會出現的排版跑掉 |
| **S4 / Trivial** | 瑕疵 | 錯字、像素級對齊誤差、不影響任何功能的視覺瑕疵 | 說明文字有錯字、圖示邊緣 1px 沒對齊 |

## 處理時限與 Release 門檻（對應 `docs/architecture-and-process.md` 的 Release Review Gate）

| 等級 | 是否阻擋里程碑 / release | 建議處理時限 |
|------|------------------------|-------------|
| S1 / Critical | **一律阻擋**，不得帶病上線 | 發現後立即處理，不排入一般待辦 |
| S2 / Major | 阻擋，除非 Producer 與使用者明確同意延後並記錄理由 | 該里程碑結束前修復 |
| S3 / Minor | 不阻擋，排入待辦追蹤 | 下個里程碑或有餘力時處理 |
| S4 / Trivial | 不阻擋，資源允許再修 | 無強制時限 |

## 回報格式（所有 QA Agent 一律附上 `severity` 欄位）

```yaml
bug_report:
  id: "BUG-001"
  severity: "S2"          # S1 | S2 | S3 | S4，依上表判斷
  found_by: "functional-tester"   # 或 balance-tester / performance-tester / usability-tester
  title: "老虎機 Spin 按鈕連點會觸發雙重扣款"
  repro_steps: ["1. 進入遊戲主畫面", "2. 快速連點 Spin 按鈕 2 次"]
  expected: "只扣款一次並開始一次 Spin"
  actual: "扣款兩次，Spin 動畫只播一次"
  environment: "Unity 2023.2 / Android 實機"
  workaround: "無" # 或描述 workaround
```

## 使用規則

1. **`functional-tester` / `usability-tester`**：回報任何問題時，一律先依上表判斷等級，不確定時傾向標高一級並說明理由，讓 `qa-lead` 覆核降級，而不是自行低估。
2. **`balance-tester` / `performance-tester`**：數值偏離或效能未達標若會實質影響玩家體驗或公平性（例如 RTP 明顯偏離目標、FPS 遠低於平台可玩門檻），一併標注對應等級；單純「建議微調」的偏差可不標等級。
3. **`qa-lead`**：彙整品質報告時，依 S1/S2 數量判斷 go/no-go；任何未解決的 S1 一律 no-go。
4. 等級判斷有爭議時，`qa-lead` 有最終分級權，但要說明理由，不要憑感覺覆蓋。
