# MCP 整合詳解

> 這是 [Kiro Multi-Agent Game Studio](../README.md) 的深入文件之一。完整索引見 README 的「深入文件（Reference）」。

## Blender MCP 整合詳解


> 本節整併自原 `blender/README.md`，是本專案目前**唯一已完成連線**的 MCP 工具說明。

### 概述

Blender MCP 是一個輕量級的 MCP Server，提供自然語言介面與 Blender Python API 互動，讓 Agent 可以探索、理解、修改複雜的場景設定。

架構流程：
```
Kiro ⇐ MCP/stdio ⇒ blender-mcp ⇐ TCP socket ⇒ Blender Add-on
```

> ⚠️ **安全警告**：MCP Server 會在 Blender 中執行 LLM 生成的程式碼，沒有任何防護措施保護你的資料。建議使用虛擬機或不含敏感資料的系統操作。

### 前置需求

- [Blender 5.1](https://www.blender.org/download/) 或更新版本
- [uv](https://docs.astral.sh/uv/) Python 套件管理工具
- [Kiro](https://kiro.dev/) 作為 LLM Client

### 安裝步驟

只需要兩個元件：**Add-on**（裝在 Blender 裡）和 **MCP Server**（由 Kiro 自動管理）。

#### 1. 安裝 uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

#### 2. Clone blender_mcp

```bash
git clone https://projects.blender.org/lab/blender_mcp.git
```

#### 3. Blender Add-on 安裝

Add-on 讓 MCP Server 能與正在運行的 Blender 實例溝通。

**方法 A：拖放安裝（推薦）**
- 從 [Release 頁面](https://projects.blender.org/lab/blender_mcp/releases) 下載最新 `.zip`
- 拖放到 Blender 視窗中（需拖放兩次：第一次加入 Blender Lab repository，第二次安裝 add-on）
- 此方法可在新版本可用時收到更新通知

**方法 B：手動安裝**
- 下載 [mcp-1.0.0.zip](https://projects.blender.org/lab/blender_mcp/releases/download/v1.0.0/mcp-1.0.0.zip)
- Blender → Edit → Preferences → Extensions → [Install from Disk](https://docs.blender.org/manual/en/latest/editors/preferences/extensions.html#install)

#### 4. Kiro MCP 設定（stdio 模式）—— 本專案已完成此步

`.kiro/settings/mcp.json`：

```json
{
  "mcpServers": {
    "blender-mcp": {
      "command": "uv",
      "args": ["--directory", "/Users/dayho/Documents/blender_mcp/mcp", "run", "blender-mcp"],
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Kiro 會自動啟動並管理 blender-mcp process，不需要手動開 terminal。**每次要用 `art/blender-team` 前，記得先開啟 Blender 並確認 add-on 已啟用**，否則連線會失敗（`blender-team.md` 會先自檢連線狀態，連不上會直接回報，不會嘗試繼續建模）。

### 可用 Tools

MCP Server 提供以下工具供 Agent 使用（`art/blender-team` 的工作流程會用到大部分）：

| 工具名稱 | 說明 |
|---------|------|
| `execute_blender_code` | 在連線的 Blender 中執行 Python 程式碼 |
| `get_blendfile_summary_datablocks` | 取得 blend 檔案摘要：data-block 數量、workspace、render engine |
| `get_blendfile_summary_missing_files` | 報告缺失的外部檔案參考 |
| `get_blendfile_summary_of_linked_libraries` | 回傳連結的 library 檔案樹狀結構 |
| `get_blendfile_summary_path_info` | 取得 blend 檔案路徑、儲存狀態等資訊（blender-team 用此工具做連線自檢） |
| `get_blendfile_summary_usage_guess` | 猜測 blend 檔案的主要用途 |
| `get_object_detail_summary` | 回傳指定物件的結構化摘要 |
| `get_objects_summary` | 回傳場景的 collection 階層和物件 |
| `get_python_api_docs` | 取得 Blender Python API 文件 |
| `get_screenshot_of_area_as_image` | 擷取 Blender 單一區域截圖 |
| `get_screenshot_of_window_as_image` | 擷取整個 Blender 視窗截圖 |
| `get_screenshot_of_window_as_json` | 回傳 Blender 視窗佈局的 JSON 描述 |
| `jump_to_tab_by_name` | 切換 workspace tab |
| `jump_to_view3d_object_by_name` | 3D viewport 聚焦到指定物件 |
| `render_thumbnail_to_path` | 渲染低品質縮圖（blender-team 用於產出確認縮圖） |
| `render_viewport_to_path` | 使用目前設定渲染場景 |

### 使用範例

以下為官方文件提供的 prompt 範例，可直接對 `art/blender-team` 或原生對話使用：

```
Analyze the scene and list the outliers: objects with highest polygon count
but smaller size from the camera point of view.
```

```
With the current open Blender file fix the name of all the data-blocks
to remove typos. Report back which data-blocks got fixed.
```

```
With the current open Blender file suggest descriptive names for all
data-blocks, and apply if approved.
```

```
With the current open Blender file explain what the main geometry nodes
setup is doing. Add inline documentation for it with frame elements.
Create a Text data-block with the result of analysis.
```

其他可探索的 prompt：
- 將所有 data-block 從法文翻譯成英文
- Mesh 沒被 armature 變形，如何修復？
- Blender 渲染時記憶體不足，如何優化？
- 找出有不良法線的物件
- 檢查非均勻變換的 mesh 物件
- 設定 compositing nodes 以同時儲存 SDR 和 HDR 圖片
- 匯出的影片無法在瀏覽器播放，該調整哪些設定？

### 參考資料

- [Blender MCP 官方頁面](https://www.blender.org/lab/mcp-server/#llm-client)
- [blender_mcp Releases](https://projects.blender.org/lab/blender_mcp/releases)
- [blender_mcp 原始碼 Repo](https://projects.blender.org/lab/blender_mcp)
- [MCP 協議說明](https://modelcontextprotocol.io/)
- [uv 安裝指南](https://docs.astral.sh/uv/getting-started/installation/)

---

## ComfyUI MCP 整合詳解


> 依 [Comfy 官方 Agent Tools / MCP 文件](https://docs.comfy.org/agent-tools) 整理。官方目前有四條路：**Comfy Cloud MCP**（一級支援、需訂閱）、**Comfy Local MCP**（官方第一方 local server，**目前私測中，尚未公開**）、**Comfy Partner MCP**（呼叫 30+ 夥伴模型 API，非本機 workflow）、**社群 local MCP server**（自架、免費）。因為官方第一方 local server 還申請不到，本專案選擇社群方案中功能最完整、維護最活躍的 [`artokun/comfyui-mcp`](https://github.com/artokun/comfyui-mcp)。

### 為什麼選 `artokun/comfyui-mcp`

| 方案 | 特性 |
|------|------|
| [Comfy Cloud MCP](https://docs.comfy.org/agent-tools/cloud) | 官方一級支援、免本機 GPU，但需要訂閱與 credits，走遠端 HTTPS |
| [Comfy Local MCP](https://docs.comfy.org/agent-tools/local)（`comfy-local-mcp`） | 官方第一方，但**私測中，一般人目前申請不到** |
| [Comfy CLI](https://docs.comfy.org/agent-tools/comfy-cli)（`comfy generate`） | 終端機指令，不是 MCP tool，適合腳本/CI 而非對話式操作 |
| [`lalanikarim/comfy-mcp-server`](https://github.com/lalanikarim/comfy-mcp-server) | 本專案先前採用，僅 2 個工具且綁定單一 workflow JSON，功能過於陽春 |
| [`joenorton/comfyui-mcp-server`](https://github.com/joenorton/comfyui-mcp-server) | 功能較完整，但需另開常駐 HTTP server process |
| [`artokun/comfyui-mcp`](https://github.com/artokun/comfyui-mcp) | **本專案採用**：108 個工具、持續維護中（近期仍有更新）、`npx` 一鍵啟動並自動偵測本機 ComfyUI 路徑與 port，不需手動指定 workflow JSON |

### 設定內容（`.kiro/settings/mcp.json`）

```json
{
  "mcpServers": {
    "comfyui": {
      "command": "npx",
      "args": ["-y", "comfyui-mcp"],
      "env": {
        "CIVITAI_API_TOKEN": "",
        "HUGGINGFACE_TOKEN": ""
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

- 不需要手動填 `COMFY_URL`：server 會自動偵測本機 ComfyUI（先試 port 8188，CLI 預設；再試 8000，Desktop App 預設）
- 不需要手動填 workflow JSON 路徑或 node ID：`generate_image` 等高階工具會自動建構 workflow
- `CIVITAI_API_TOKEN` / `HUGGINGFACE_TOKEN` 為選用，只有在需要從這兩個平台下載模型時才需要填
- 若你的 ComfyUI 裝在非標準路徑，可額外設定 `COMFYUI_PATH` 環境變數指向資料目錄

### 可用 Tools（108 個，依用途分組，詳見「團隊角色與職責」章節的完整表）

| 分組 | 代表工具 |
|------|---------|
| 高階生成 | `generate_image`, `generate_with_controlnet`, `generate_with_ip_adapter`, `generate_audio` |
| 資產與迭代 | `view_image`, `regenerate`, `list_assets`, `analyze_color` |
| Workflow 組裝 | `create_workflow`, `modify_workflow`, `validate_workflow` |
| 模型管理 | `search_models`, `download_model`, `list_local_models` |
| 診斷 | `get_logs`, `get_history`, `clear_vram` |

> 完整清單見 [artokun/comfyui-mcp 文件](https://comfyui-mcp.artokun.io/docs)。

### 安全提醒

- Server 預設綁定本機，不要在沒有額外驗證的情況下對外公開（若要遠端存取，該專案有 `--tunnel` 模式會自動產生 auth token，見其文件）
- `mcp.json` 中若填入任何 API Key（`CIVITAI_API_TOKEN`、`HUGGINGFACE_TOKEN` 等），應改用環境變數而非寫死在檔案中，並確認該檔案已被 `.gitignore` 排除

### 參考資料

- [Comfy 官方 Agent Tools / MCP 文件](https://docs.comfy.org/agent-tools)
- [Comfy Local MCP](https://docs.comfy.org/agent-tools/local)（官方第一方，私測中）
- [Comfy Cloud MCP](https://docs.comfy.org/agent-tools/cloud)
- [`artokun/comfyui-mcp`](https://github.com/artokun/comfyui-mcp)（本專案採用，MIT License，331+ stars）

---

## Unity MCP 整合詳解


> `engineering/unity-team` 透過開源專案 [CoplayDev/unity-mcp](https://github.com/CoplayDev/unity-mcp) 操作 Unity Editor。

### 設定內容（`.kiro/settings/mcp.json`）

```json
{
  "mcpServers": {
    "unity-mcp": {
      "url": "http://127.0.0.1:8080/mcp",
      "transport": "http",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Unity 端安裝了 [CoplayDev/unity-mcp](https://github.com/CoplayDev/unity-mcp) UPM 套件（Package Manager → Add package from git URL → `https://github.com/CoplayDev/unity-mcp.git?path=/MCPForUnity#main`），並在 Window → MCP for Unity 視窗 Start Server 後，就會在這個 port 上監聽。若視窗顯示的 port 不是 8080，同步修改 `url` 即可。

### 連線方式：HTTP（預設）vs stdio（備援）

| 方式 | 使用情境 | 設定 |
|------|---------|------|
| HTTP（本專案採用） | 預設、推薦，只要 port 沒被佔用就用這個 | `{ "url": "http://127.0.0.1:8080/mcp", "transport": "http" }` |
| stdio | HTTP port 被佔用或有防火牆問題時的備援 | `{ "command": "uvx", "args": ["unity-mcp"], "transport": "stdio" }`（需先安裝 [uv](https://docs.astral.sh/uv/)） |

> **安全提醒**：HTTP 是刻意的設計選擇——這個 endpoint 只跟本機 `localhost`（loopback）上的 Unity Editor 通訊，流量不會離開這台機器，所以不需要 HTTPS。不要把這個 port 對外公開監聽。

### 可用 Tools（概念分組，詳細參數依 `unity-mcp` 版本而定）

| 分組 | 工具 |
|------|------|
| 資產/材質 | `manage_asset`, `manage_material`, `manage_texture`, `manage_shader` |
| 場景/物件 | `manage_scene`, `manage_gameobject`, `manage_components`, `manage_prefabs` |
| UI/視覺 | `manage_ui`, `manage_camera`, `manage_animation`, `manage_graphics` |
| 專案/編輯器 | `manage_packages`, `manage_editor`, `manage_script`, `create_script` |
| 執行/查詢 | `run_tests`, `read_console`, `batch_execute`, `find_gameobjects` |
| 只讀 Resource | `project_info`, `editor_state`, `gameobject`, `editor_selection` |

### `unity-team` 遵循的核心紀律

- **連線健康檢查優先**：任何操作前先讀取 `project_info`，失敗就停下回報，不猜測、不硬闖
- **Steering-First**：場景搭建、資產批次設定、Build、效能分析、架構檢查、平台相容性檢查等任務，各有一套「先查規範再動手」的具體工具呼叫順序（完整對照表見 `unity-team.md` 的「依任務領域查對應規範」）
- **Play Mode 保護**：絕不在 Play Mode 下對場景做永久性修改
- **批次操作要有摘要**：`batch_execute` 或大量物件操作後，回報「成功 N、失敗 M」，失敗要列原因

### 參考資料

- [CoplayDev/unity-mcp](https://github.com/CoplayDev/unity-mcp)（實際執行層，MIT License，12k+ stars）
- [MCP 協議說明](https://modelcontextprotocol.io/)

---

## Godot MCP 整合詳解


> `engineering/godot-team` 透過開源專案 [Coding-Solo/godot-mcp](https://github.com/Coding-Solo/godot-mcp)（npm: `@coding-solo/godot-mcp`，4.6k+ stars）操作 Godot Editor。

### 設定內容（`.kiro/settings/mcp.json`）

```json
{
  "mcpServers": {
    "godot-mcp": {
      "command": "npx",
      "args": ["-y", "@coding-solo/godot-mcp"],
      "env": {
        "GODOT_PATH": "/Applications/Godot.app/Contents/MacOS/Godot",
        "DEBUG": "false"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

### 安裝步驟

`Coding-Solo/godot-mcp` 走 `npx` 自動安裝，**不需要手動 clone + build**：

1. 確認已安裝 Node.js（≥18）與 Godot。
2. `mcp.json` 已設好 `npx -y @coding-solo/godot-mcp`（見上方設定內容），Kiro 會自動下載並啟動這個 MCP Server。
3. 把 `GODOT_PATH` 換成你本機 Godot 執行檔路徑（macOS 預設 `/Applications/Godot.app/Contents/MacOS/Godot`；若 Godot 已在系統 PATH 裡可省略此變數，會自動偵測）。
4. 儲存後 Kiro 會自動嘗試連線。

> 想改用原始碼建置（可選）：`git clone https://github.com/Coding-Solo/godot-mcp && cd godot-mcp && npm install && npm run build`，再把 `command`/`args` 改成 `"node"` + `["/path/to/godot-mcp/build/index.js"]`。

### 可用 Tools

| 分組 | 工具 |
|------|------|
| 系統 | `get_godot_version` |
| 專案 | `launch_editor`, `run_project`, `stop_project`, `list_projects`, `get_project_info` |
| 場景 | `create_scene`, `add_node`, `edit_node`, `remove_node`, `load_sprite`, `export_mesh_library`, `save_scene` |
| 除錯 | `get_debug_output` |
| UID（4.4+） | `get_uid`, `update_project_uids` |

### `godot-team` 遵循的核心紀律

- **GDScript 一律加型別標註**：不生成未標型別的變數/參數/回傳值
- **Composition 優先於深層 Nesting**：避免超過 10 層節點階層
- **Signal 優先於直接呼叫**：跨節點溝通走事件驅動，全域事件用 Autoload 的 Event Bus
- **`run_project` 會阻塞直到遊戲視窗關閉**：測試用途改用 `stop_project` 中斷，不要當成錯誤重試
- **UID 工具只支援 4.4+**：較舊版本改用 `res://` 路徑

### 參考資料

- [Coding-Solo/godot-mcp](https://github.com/Coding-Solo/godot-mcp)（實際執行層，MIT License，npm: `@coding-solo/godot-mcp`）
- [Godot 官方文件](https://docs.godotengine.org/)

---

## Unreal MCP 整合詳解


> `engineering/unreal-team` 透過開源專案 [flopperam/unreal-engine-mcp](https://github.com/flopperam/unreal-engine-mcp) 的**開源 local MCP**（`Python/` 資料夾 + `UnrealMCP` C++ 外掛）操作 Unreal Editor，非付費 Hosted 版。

### 為什麼選 local MCP 而非 Hosted Flop MCP

| 方案 | 特性 |
|------|------|
| Hosted Flop MCP（`agent.flopperam.com/mcp`） | 50+ 工具（Blueprint 全生命週期、Niagara VFX、GAS、Sequencer 等），但需付費 API Key，走遠端 |
| **Local MCP（本專案採用）** | 免費、開源、走本機 stdio；工具集較小（場景操作、Actor 管理、基礎 Blueprint、World Building） |

若後續需要 Hosted 版的進階能力，可在 `unreal-team.md` 裡明確告知使用者這個限制並詢問是否要切換。

### 設定內容（`.kiro/settings/mcp.json`）

```json
{
  "mcpServers": {
    "unreal-engine": {
      "command": "uv",
      "args": [
        "--directory",
        "/ABSOLUTE/PATH/TO/unreal-engine-mcp/Python",
        "run",
        "unreal_mcp_server_advanced.py"
      ],
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

### 安裝步驟

1. Clone 到專案外的固定位置（不要放進 Unreal 專案內）：
   ```bash
   cd ~/Desktop
   git clone https://github.com/flopperam/unreal-engine-mcp.git
   ```
2. 把 `UnrealMCP` 外掛複製到 Unreal 專案的 `Plugins/` 資料夾（在專案根目錄，即 `.uproject` 所在位置執行）：
   ```bash
   cp -r ~/Desktop/unreal-engine-mcp/UnrealMCP Plugins/
   ```
3. 對 `.uproject` 按右鍵 → Generate Visual Studio/Xcode project files → 開啟並 Build（Development Editor）
4. Unreal Editor → Edit → Plugins → 搜尋 `UnrealMCP` → 啟用 → 重啟 Editor
5. 安裝 Python 3.12+ 與 [uv](https://docs.astral.sh/uv/)
6. 把 `mcp.json` 的路徑換成你本機 `unreal-engine-mcp/Python` 的絕對路徑

### 可用 Tools（local MCP 版）

| 分組 | 工具 |
|------|------|
| Blueprint 視覺化腳本 | `add_node`, `connect_nodes`, `delete_node`, `create_variable`, `create_function` 等 |
| Blueprint 分析 | `read_blueprint_content`, `analyze_blueprint_graph` 等 |
| Blueprint 系統 | `create_blueprint`, `compile_blueprint`, `add_component_to_blueprint` |
| World Building | `create_town`, `construct_house`, `create_tower`, `create_castle_fortress` 等 |
| 物理與材質 | `spawn_physics_blueprint_actor`, `apply_material_to_actor` 等 |
| Actor 管理 | `get_actors_in_level`, `find_actors_by_name`, `set_actor_transform` |

### 已知問題（`unreal-team` 已內建規避邏輯）

- **絕對不要用 `ce` console command**：透過 MCP 執行會導致 Unreal Editor 立即 crash
- **`set_component_property` 設定 `OverrideMaterials` 不可靠**：改用已驗證的 Blueprint SCS 做法（見 `unreal-team.md` 內文）
- **避免大量 Undo**：批次還原優先用明確重新套用，而非連續 40+ 次 undo

### 參考資料

- [flopperam/unreal-engine-mcp](https://github.com/flopperam/unreal-engine-mcp)（實際執行層，MIT License，1.1k+ stars）
- [Unreal Engine 官方文件](https://dev.epicgames.com/documentation/en-us/unreal-engine/)

---

## Cocos MCP 整合詳解


> `engineering/cocos-team` 透過社群外掛 [DaxianLee/cocos-mcp-server](https://github.com/DaxianLee/cocos-mcp-server) 操作 Cocos Creator Editor。特別適合輕量跨平台/H5 遊戲，包含老虎機這類需要快速多平台部署的類型。

### 設定內容（`.kiro/settings/mcp.json`）

```json
{
  "mcpServers": {
    "cocos-creator": {
      "url": "http://127.0.0.1:3000/mcp",
      "transport": "http",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

### 安裝步驟

1. 下載 [`cocos-mcp-server`](https://github.com/DaxianLee/cocos-mcp-server) 或從 [Cocos Store](https://store.cocos.com/app/detail/7941) 安裝
2. 複製整個 `cocos-mcp-server` 資料夾到 Cocos Creator 專案的 `extensions/cocos-mcp-server/`
3. `cd extensions/cocos-mcp-server && npm install && npm run build`
4. 重啟 Cocos Creator 或刷新擴展
5. 擴展 → Cocos MCP Server → 開啟面板 → 設定 port（預設 3000）→ 點擊「啟動伺服器」
6. 若 port 不是 3000，同步修改 `mcp.json` 的 `url`

### 可用 Tools（依用途分組）

| 分組 | 工具 |
|------|------|
| 場景 | `scene_get_current_scene`, `scene_create_scene`, `scene_open_scene`, `scene_save_scene` |
| 節點 | `node_create_node`, `node_find_node_by_name`, `node_get_all_nodes`, `node_set_node_transform` |
| 元件 | `component_add_component`, `component_set_component_property`, `component_attach_script` |
| Prefab | `prefab_create_prefab`, `prefab_instantiate_prefab`, `prefab_get_prefab_list` |
| 專案 | `project_get_project_info`, `project_build_project`, `project_run_project` |
| 除錯 | `debug_get_console_logs`, `debug_get_performance_stats`, `debug_validate_scene` |
| 進階資產 | `advancedAsset_batch_import_assets`, `advancedAsset_get_unused_assets` |

### `cocos-team` 遵循的核心紀律

- **`node_create_node` 一定先取得 `parentUuid`**：否則節點會建到場景根節點
- **`component_set_component_property` 一定要明確指定 `propertyType`**：省略會靜默失敗
- **資產路徑一律用 `db://` 前綴**：不要用檔案系統絕對路徑
- **2D/3D 節點座標欄位不同**：2D 只用 x/y，3D 用完整 x/y/z

### 參考資料

- [DaxianLee/cocos-mcp-server](https://github.com/DaxianLee/cocos-mcp-server)（實際執行層，1.2k+ stars）
- [Cocos Creator 官方文件](https://docs.cocos.com/creator/manual/en/)

---

## Figma MCP 整合詳解


> `design/ui-ux-team` 透過 [官方 Figma MCP Server](https://developers.figma.com/docs/figma-mcp-server/) 讀取設計檔的版面與 Design Token，產出 handoff 規格交給對應引擎 Team。Kiro 是 Figma 官方文件明列支援的 MCP client（支援 Desktop / Remote / write-to-canvas / Kiro Powers）。

### Figma 在遊戲開發負責哪個層面

以遊戲開發者的角度，Figma 不碰 3D 資產、不碰遊戲邏輯，它負責的是**介面與使用者體驗（UI/UX）層**：

| Figma 負責 | 具體產出 |
|-----------|---------|
| UX 流程 | 資訊架構、畫面流程（主選單 → 遊戲 → 設定 → 商店）、導覽、新手引導、互動原型 |
| UI 版面 | HUD（血條/彈藥/分數）、選單、彈窗、設定頁、商店；老虎機的 reel frame / spin 按鈕 / paytable / 中獎表演 UI 的排版 |
| Design System | 色彩、字型階層、間距、圓角、按鈕狀態（normal/hover/pressed/disabled） |
| Handoff | 版面座標、尺寸、間距、顏色、切圖清單，交接給引擎 Team 在原生 UI 系統重建 |

**分工界線**：3D 模型 / PBR 貼圖交 `art/blender-team`、`art/comfyui-team`；遊戲邏輯交對應引擎 Team；像素素材（icon/按鈕/背景）由 `art/comfyui-team` 生成——Figma 只負責「怎麼排、怎麼流動、用什麼 Token」，`ui-ux-team` 出規格，引擎 Team 在遊戲引擎的 UI 系統（Unity UI Toolkit / Godot Theme / Unreal UMG / Cocos UI）實作。

### 三種接法（本專案預設官方 Remote）

| 接法 | endpoint / 指令 | 需求 | 適用 |
|------|----------------|------|------|
| **官方 Remote（本專案採用）** | `https://mcp.figma.com/mcp`（HTTP，OAuth） | 任何 Figma 方案/席次；首次於 Kiro 完成 OAuth 授權 | 首選，支援 write-to-canvas |
| 官方 Desktop | `http://127.0.0.1:3845/mcp`（HTTP） | Figma 桌面 App + 付費方案的 **Dev/Full 席次** | 企業/組織特定情境 |
| 社群 Framelink | `npx -y figma-developer-mcp --figma-api-key=<TOKEN> --stdio` | Figma Personal Access Token（MIT，走 REST API 讀取） | 不想走 OAuth、只需讀版面時 |

### 設定內容（`.kiro/settings/mcp.json`）

```json
{
  "mcpServers": {
    "figma": {
      "url": "https://mcp.figma.com/mcp",
      "transport": "http",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

若改用社群 Framelink（需自備 token，建議走環境變數）：

```json
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["-y", "figma-developer-mcp", "--figma-api-key=${FIGMA_TOKEN}", "--stdio"],
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

### 使用方式

1. 在 Figma Design 選取要實作的 frame / layer → 右鍵 **Copy link to selection**
2. 切到 `design/ui-ux-team`，貼上連結並描述需求（MCP 會從 URL 解析 node ID）
3. `ui-ux-team` 讀取版面與 Design Token，整理成 handoff 規格（版面座標、尺寸、Token、切圖清單）
4. 需要裝飾性 UI 素材時，把切圖清單交給 `art/comfyui-team` 生成，再回填版面
5. 把 handoff 規格交給對應引擎 Team，在其原生 UI 系統重建

### 可用 Tools（官方 Server，依用途分組）

| 分組 | 能力 |
|------|------|
| 讀取設計 context | 取得選取 frame/node 的尺寸、間距、階層、auto-layout |
| 萃取 Design Token | 取得 variables / styles（色彩/字型/間距，handoff 核心） |
| 產生程式碼 | 從選取 frame 產生 UI 程式碼（供引擎 Team 參考，非最終產物） |
| 取得圖片 | 匯出 frame/node 為圖片（供標注與切圖） |
| Code Connect | 對應設計元件 ↔ 程式元件，讓產出與 codebase 一致 |
| 寫回畫布（Remote） | 依 design system 建立 frame / component / variable |

> 實際工具名稱依接法（Remote / Desktop / Framelink）與版本而定，呼叫前先確認精確名稱。完整清單見 [Tools and prompts](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/)。

### 安全提醒

- 官方 Remote 走 OAuth，不需在 `mcp.json` 寫 token；若改用 Framelink，`FIGMA_TOKEN` 屬敏感資訊，應走環境變數而非寫死，並確認存放金鑰的檔案已被 `.gitignore` 排除
- Desktop Server 只監聽本機 loopback（`127.0.0.1:3845`），不要對外公開

### 參考資料

- [Figma MCP Server 開發者文件](https://developers.figma.com/docs/figma-mcp-server/)
- [Guide to the Figma MCP server（含支援的 client 列表，Kiro 在列）](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server)
- [Remote Server 安裝](https://help.figma.com/hc/en-us/articles/35281350665623) / [Desktop Server 安裝](https://help.figma.com/hc/en-us/articles/35281186390679)
- [GLips/Figma-Context-MCP（社群 Framelink，MIT）](https://github.com/GLips/Figma-Context-MCP)

---

## GitHub MCP 整合詳解


> `producer`（及 Small Team 規模的協作）透過官方 [GitHub MCP Server](https://github.com/github/github-mcp-server) 讀寫 GitHub 的 issues、Pull Request 與 **Projects 看板**，作為原願景中 Linear 的替代方案。好處：任務追蹤與程式碼同源（都在你的 GitHub repo），少一個外部帳號。

### 設定內容（`.kiro/settings/mcp.json`，原生 binary、免 Docker）

```json
{
  "mcpServers": {
    "github": {
      "command": "github-mcp-server",
      "args": ["stdio"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "" },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

### 安裝步驟（原生 binary，最輕量、免 Docker/Copilot）

1. 下載對應平台的執行檔：[github-mcp-server releases](https://github.com/github/github-mcp-server/releases)（或有 Go 環境時 `go install github.com/github/github-mcp-server/cmd/github-mcp-server@latest`）。
2. 把 `github-mcp-server` 放進 `PATH`（macOS 可放 `/usr/local/bin`，或 `~/go/bin` 已在 PATH）。
3. 建立 GitHub Personal Access Token（PAT）：GitHub → Settings → Developer settings → Personal access tokens，授予任務追蹤所需權限（至少 repo / issues / projects）。
4. 把 PAT 填入 `GITHUB_PERSONAL_ACCESS_TOKEN`（**建議用環境變數注入，不要把 token commit 進 repo**）。
5. 儲存後 Kiro 會自動啟動並管理這個 MCP Server（stdio）。

### 三種接法（依輕量程度）

| 接法 | 設定 | 需求 | 適用 |
|------|------|------|------|
| **原生 binary（本專案採用，最輕量）** | `command: "github-mcp-server"`, `args: ["stdio"]` + `GITHUB_PERSONAL_ACCESS_TOKEN` | 單一執行檔，**免 Docker、免 Copilot** | 想輕量、本地控制 |
| 官方 Remote | `https://api.githubcopilot.com/mcp/`（Streamable HTTP，PAT/OAuth） | 零本機安裝，但需 **GitHub Copilot 授權** | 有 Copilot、想完全免安裝 |
| 本機 Docker | `ghcr.io/github/github-mcp-server` + PAT | 需 Docker daemon（較重） | 已慣用 Docker 的環境 |

### 能做什麼（用途分組）

| 分組 | 能力 |
|------|------|
| Projects 看板 | 讀寫 Project 項目、欄位、狀態（任務追蹤核心，取代 Linear） |
| Issues | 建立/查詢/更新/關閉 issue、標籤、指派 |
| Pull Requests | 建立/檢視/審查 PR、留言 |
| Repo / Code | 讀取檔案、搜尋程式碼、瀏覽分支 |

> 工具數量很多、會佔不少 context。必要時用本機的 `--toolsets` 或 Remote 的 `X-MCP-Toolsets` 只開啟需要的工具集（例如只留 `issues` + `projects`）。

### 安全提醒

- **PAT 是高權限憑證**：只授予必要範圍，不要寫進會被 commit 的檔案，優先用環境變數注入（`.gitignore` 已排除常見祕密檔）。
- 原生 binary / Docker 方式流量都在本機與 GitHub API 之間；Remote 方式走 GitHub 官方 endpoint。

### 參考資料

- [github/github-mcp-server](https://github.com/github/github-mcp-server)（官方，MIT License）
- [GitHub Projects 官方文件](https://docs.github.com/issues/planning-and-tracking-with-projects)

---
