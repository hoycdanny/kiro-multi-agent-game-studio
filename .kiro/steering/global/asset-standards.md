---
inclusion: always
---

# 資產標準（Global）

所有 3D / 美術資產產出前，Agent 必須遵守以下規範。

## 命名規範

```
{asset_type}_{name}_{version}
```

範例：`weapon_sword_01`

- `asset_type`：`weapon` / `character` / `prop` / `environment` / `texture` / `sprite` / `symbol`（老虎機符號）/ `ui` / `rig` / `anim` / `sfx` / `music` / `voice` / `locale`
- `name`：資產名稱（小寫、底線分隔）
- `version`：整數，從 `01` 開始

## 3D 模型規範（Blender Team）

| 項目 | 規則 |
|------|------|
| Poly Budget | 依道具類型：小道具 ≤ 3,000 / 角色 ≤ 8,000 / 場景大型物件 ≤ 15,000 |
| Scale | 匯出前確認 Blender unit = 1m，對應 Unity import scale 0.01 |
| UV | 必須展開，不可有重疊面（除非刻意鏡射） |
| Origin | 物件原點需在底部中心或邏輯上合理的位置（武器類在握把處） |
| Collider Mesh | 若複雜度高，需另外建立簡化版 collider mesh |

## 產出後回報格式

每次產出資產，回報時需附上：
- 資產全名（依命名規範）
- 實際 poly 數
- 是否有 UV 展開
- 匯出格式（`.fbx` / `.glb`）與檔案路徑

## 資產落地目錄（所有 Team 交付物的存放位置）

各 Team 產出的中介資產一律落在 repo 根目錄的 `assets/`，讓上下游有明確交接路徑（完整說明見 `assets/README.md`）：

| 產出 Team | 落地目錄 | 內容 |
|-----------|----------|------|
| `comfyui-team` | `assets/concept/` `assets/textures/` `assets/sprites/` `assets/ui/` | 概念圖、PBR 貼圖、sprite/符號、UI 切圖 |
| `blender-team` | `assets/models/` | 靜態 3D 模型（.fbx/.glb） |
| `animator` | `assets/rigs/` `assets/animations/` | 骨架、動畫 clip |
| `audio-team` | `assets/audio/sfx/` `assets/audio/music/` `assets/audio/voice/` | 音效、音樂、配音 |
| `localization-team` | `assets/locales/` | 多語 locale 檔 |
| `balance-tester` | `assets/sim/` | 模擬腳本與 RTP/經濟平衡報告 |

規則：
- 檔名一律用命名規範 `{asset_type}_{name}_{version}`，依 `asset_type` 分目錄，不另開子資料夾。
- 所有二進位資產走 Git LFS（見根目錄 `.gitattributes`），首次使用先 `git lfs install`。
- `assets/` 放中介來源資產；匯入引擎後的引擎專屬檔（.meta/.import/.uasset）由各引擎專案自行管理，不放這裡。

## 音訊規範（Audio Team）

| 項目 | 規則 |
|------|------|
| 格式 | 母帶用 `.wav`（無損）交付；壓縮版（.ogg/.mp3）依引擎需求另出 |
| 取樣率 | SFX/語音 48kHz、音樂 44.1kHz 起，交付時標注 |
| 響度 | 標注 LUFS 或峰值，避免各音效音量落差過大 |
| 迴圈 | BGM 若需無縫循環，標注 loop point |

## 動畫規範（Animator）

| 項目 | 規則 |
|------|------|
| 骨架 | 命名一致、階層乾淨；人形建議相容 Humanoid retarget |
| 動畫 | 每個 clip 標注 frame range、fps、是否 loop、root motion 與否 |
| 匯出 | Unity/Unreal 常用 `.fbx`；Godot/Cocos 常用 `.glb`，依目標引擎 |
