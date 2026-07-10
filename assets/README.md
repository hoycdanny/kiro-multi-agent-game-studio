# assets/ — 團隊交付物落地目錄

各 Team 產出的資產統一落在這裡，讓 Pipeline 上下游有**明確、可預期**的交接路徑。命名規範見 `.kiro/steering/global/asset-standards.md`（`{team_id}.{asset_type}_{name}_{version}`）。

## 目錄結構

```
assets/
├── concept/       # ComfyUI Team：概念圖 / 風格探索
├── textures/      # ComfyUI Team：PBR 貼圖（_albedo / _normal / _roughness）
├── sprites/       # ComfyUI Team：2D sprite、老虎機符號（symbol）
├── ui/            # ComfyUI Team：UI 切圖素材（依 ui-ux-team 的切圖清單）
├── models/        # Blender Team：靜態 3D 模型（.fbx / .glb）
├── rigs/          # Animator：骨架 / 綁定（rigged 模型）
├── animations/    # Animator：動畫 clip（.fbx / .glb，含動作）
├── audio/
│   ├── sfx/       # Audio Team：音效
│   ├── music/     # Audio Team：背景音樂 / BGM
│   └── voice/     # Audio Team：配音 / 語音
├── locales/       # Localization Team：多語 locale 檔（.po / .csv / .json）
└── sim/           # Balance Tester：模擬腳本與報告（RTP / 經濟平衡）
```

> 每個 `<team_id>` 的資產以檔名前綴區分（例如 `vt_001.symbol_seven_01.png`），不另開 per-team 子資料夾，避免路徑爆炸；要隔離多團隊時用檔名前綴即可。

## Pipeline 交接對照

| 產出 Team | 落地目錄 | 下游讀取者 |
|-----------|----------|-----------|
| `comfyui-team` | `concept/` `textures/` `sprites/` `ui/` | `blender-team`（貼圖）、引擎 Team（sprite/UI） |
| `blender-team` | `models/` | `animator`（要動畫時）、引擎 Team |
| `animator` | `rigs/` `animations/` | 引擎 Team |
| `audio-team` | `audio/sfx|music|voice/` | 引擎 Team |
| `localization-team` | `locales/` | 引擎 Team |
| `balance-tester` | `sim/` | `slot-game-expert` / `economy-designer`（回饋數值） |

## 注意

- 這裡放的是**中介交付物**（來源資產）；最終匯入各引擎專案後的引擎專屬檔（.meta、.import、.uasset 等）由各引擎專案自己的 repo/`.gitignore` 管理，不放這裡。
- 二進位檔一律走 Git LFS（見根目錄 `.gitattributes`），commit 前先 `git lfs install`。
