# 配音與音樂 Pipeline（Voiceover / Music）

> 這是 [Kiro Multi-Agent Game Studio](../README.md) 的深入文件之一。完整索引見 README 的「深入文件（Reference）」。
>
> 參考《The Game Production Handbook》Ch21（Voiceover）、Ch22（Music）的流程框架，但本文件**誠實區分「AI Agent 能自動化的部分」與「需要人類 Producer 線下處理的部分」**——不虛構本框架目前不具備的能力。`audio-team` 目前只能透過 ComfyUI 的音訊生成能力（`generate_audio`，ACE Step / Stable Audio 等 model family）產出音訊，沒有任何工具可以「找真人演員」「談授權合約」「進錄音室」，這些是人類的事。

## 兩條路徑

本框架下的配音/音樂製作分成兩條路徑，開工前先確認走哪一條（或混用）：

| | AI 生成路徑 | 真人製作路徑 |
|---|------------|-------------|
| 執行者 | `audio-team`（Agent） | 人類配音演員 / 作曲家（Producer 線下協調） |
| 本框架能自動化的部分 | 生成、命名、規格、落地到 `shared/audio/` | 無——僅提供清單協助人類規劃 |
| 適用情境 | 原型階段、預算有限、風格化/非寫實需求、佔位音效 | 正式上線、需要角色個性/情感表演、品牌調性要求高 |

多數專案會混用：原型/測試階段先用 AI 生成佔位，正式上線前再決定哪些角色/曲目要換成真人製作。

---

## 配音（Voiceover）Pipeline

### AI 生成路徑（`audio-team` 可執行）

1. **確認角色與台詞**：從 `narrative-designer` 或 `game-designer` 取得對話內容、角色語氣描述
2. **生成**：`audio-team` 用 `generate_audio`（依 model family 支援度，多語言/情緒控制能力有限，需先確認工具實際支援的參數）
3. **命名與交付**：依 `asset-standards.md` 命名為 `voice_{角色}_{台詞編號}_01`，落到 `shared/audio/voice/`
4. **已知限制**（誠實聲明）：目前 AI 語音生成的情感表現力、角色一致性通常不如真人演員；長台詞/複雜情緒場景建議人工複核，不要假設生成結果可直接上線不經檢查

### 真人製作路徑（人類 Producer 線下處理，本框架不介入執行）

以下步驟**沒有對應的 Agent 或 MCP 工具**，需要使用者本人或其團隊線下處理；`audio-team` 可以幫忙**整理清單**，但不能代替執行：

1. **選角（Casting）**：決定角色聲音特質，尋找/試聽配音演員 —— 人類決策
2. **合約與授權**：談配音費用、使用範圍、後續加購條款 —— 人類簽署，見下方「授權檢查清單」
3. **錄音**：排錄音室/遠端錄音、方向指導、多次 take —— 人類執行
4. **後製**：剪輔、降噪、對嘴同步（若需要）—— 人類或專業後製工具，非本框架範疇
5. **交付整合**：完成後把最終音檔放進 `shared/audio/voice/`，`audio-team` 可協助確認命名/格式是否符合 `asset-standards.md`

### 配音清單範本（`audio-team` 可協助填寫，供人類 Producer 規劃真人配音時使用）

```yaml
voiceover_plan:
  characters:
    - name: "主角"
      voice_style: "沉穩、略帶滄桑，30-40 歲男聲"
      line_count: 120
      path: "ai_generated"        # ai_generated | human_needed | human_recorded
    - name: "反派"
      voice_style: "尖銳、戲劇化"
      line_count: 45
      path: "human_needed"        # 標注為需要真人配音的角色
  notes: "反派角色情感層次較複雜，建議真人配音；主角可先用 AI 生成佔位，上線前視預算決定是否置換"
```

---

## 音樂（Music）Pipeline

### AI 生成路徑（`audio-team` 可執行）

1. **確認曲風/情緒**：讀 `.kiro/steering/project/style-guide.md`「聲音基調」章節，或向使用者確認
2. **生成**：用 `generate_audio`（ACE Step 支援 lyrics/musical key 等參數，Stable Audio 適合氛圍/環境音樂）
3. **Loop 處理**：BGM 若需無縫循環，標注 loop point（見 `asset-standards.md` 音訊規範）
4. **命名與交付**：`music_bgm_{場景}_01`，落到 `shared/audio/music/`

### 真人製作 / 授權路徑（人類 Producer 線下處理）

若決定不用 AI 生成、改用委託作曲家或採購授權音樂庫的曲目，以下同樣**不在本框架自動化範圍**：

1. **委託作曲**：找作曲家、談 brief、來回修改 —— 人類執行
2. **音樂庫採購**：挑選授權曲目、確認授權範圍 —— 人類執行，見下方「授權檢查清單」
3. **整合**：確認授權文件、把最終音檔放入 `shared/audio/music/`

### 音樂授權檢查清單（Music Licensing Checklist，人類 Producer 使用）

> AI 生成的音樂目前普遍存在著作權歸屬與訓練資料來源的法律不確定性，商業專案上線前建議諮詢法律顧問；以下清單協助人類 Producer 追蹤授權狀態，`compliance-release` 可協助整理格式但不提供法律意見。

```yaml
music_licensing:
  tracks:
    - title: "主選單主題"
      source: "ai_generated"      # ai_generated | commissioned | licensed_library | royalty_free
      license_status: "N/A（AI 生成，著作權歸屬待確認，商用前建議法律諮詢）"
    - title: "戰鬥音樂"
      source: "licensed_library"
      license_provider: "（廠商名稱）"
      license_type: "（例如：一次性買斷 / 訂閱制 / royalty-free）"
      usage_scope: "（是否含商用、是否含串流平台、地區限制）"
      proof_of_purchase: "（授權證明檔案路徑）"
  blockers: []
  disclaimer: "本清單為流程協助工具，非法律意見；音樂授權與 AI 生成內容的著作權狀態請諮詢當地法律顧問"
```

## 誰負責什麼（總結）

| 環節 | 負責方 |
|------|--------|
| AI 生成音效/音樂/配音 | `audio-team` |
| 決定走 AI 或真人路徑 | 使用者（Producer 可提供成本/品質取捨建議） |
| 選角、真人錄音、委託作曲 | 使用者本人線下處理，本框架不介入 |
| 授權/合約文件追蹤格式 | `compliance-release` 協助整理清單（非法律意見） |
| 最終音檔規範與落地 | `audio-team` 依 `asset-standards.md` 命名並確認格式 |
