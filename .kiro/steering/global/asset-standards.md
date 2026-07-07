---
inclusion: always
---

# 資產標準（Global）

所有 3D / 美術資產產出前，Agent 必須遵守以下規範。

## 命名規範

```
{team_id}.{asset_type}_{name}_{version}
```

範例：`vt_001.weapon_sword_01`

- `team_id`：專案代號，單專案可用 `vt_001`
- `asset_type`：`weapon` / `character` / `prop` / `environment`
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
