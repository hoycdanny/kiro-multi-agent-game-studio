# shared/ — 團隊交付物落地目錄（Agent 檔案共享空間）

各 Team 產出的資產統一落在這裡，讓 Pipeline 上下游有**明確、可預期**的交接路徑。命名規範見 `.kiro/steering/global/asset-standards.md`（`{asset_type}_{name}_{version}`）。

> **為什麼叫 `shared/`，不叫 `assets/`？**
> 這裡是 **AI Agent 之間的「原始素材／中介交付物」共享中轉站**（raw / source assets），不是任何遊戲引擎的專案資料夾。刻意避開 `assets` 這個字，是為了跟引擎內部的資源目錄徹底區隔——Unity 硬性規定資源放 `Assets/`、Cocos Creator 預設放 `assets/`（並以 `db://assets/...` 定址）。若這裡也叫 `assets/`，Agent 在寫檔路徑時極易和引擎專案目錄混淆。
> 流向：`comfyui-team` 生成貼圖 → 放這裡；`blender-team` 從這裡取貼圖、套到模型 → 再放回這裡；最後 `unity-team` / `godot-team` 等再把資產從 `shared/`**匯入**各自的引擎專案目錄。實際遊戲原始碼與場景不放這裡，由各引擎專案自行管理。

## 目錄結構

```
shared/
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

> 資產以 `asset_type` 分目錄、用命名規範前綴區分（例如 `symbol_seven_01.png`），不另開子資料夾，避免路徑爆炸。

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
