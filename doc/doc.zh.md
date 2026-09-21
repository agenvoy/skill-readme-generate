# readme-generate - 技術文件

> 返回 [README](./README.zh.md)

## 前置需求

- Python 3.10 或更高版本（腳本使用 `dict | None` 聯集型別語法）
- 可載入 `SKILL.md` skill 並執行 shell 指令的 agent harness
- Git（用於讀取 `git remote`、首次提交年份等資訊）

## 安裝

`<skills-dir>` 為所用 harness 掃描的 skill 目錄。

### 從 GitHub 複製

```bash
git clone https://github.com/agenvoy/skill-readme-generate.git \
    <skills-dir>/readme-generate
```

### 手動安裝

將下列檔案放置於 `<skills-dir>/readme-generate/`：

```
readme-generate/
├── SKILL.md                  # Skill 定義與流程協議
├── LICENSE
└── scripts/
    ├── analyze_project.py    # 原始碼分析腳本
    ├── setup_config.py       # 作者設定腳本
    ├── examples/             # README / doc 生成藍本
    └── licenses/             # 開源授權範本
```

安裝完成後，於 harness 中以 `/readme-generate` 呼叫即可。

## 設定

### 作者設定檔

每次執行 `/readme-generate` 都會先以 `setup_config.py check` 檢查 `~/.skill-readme-generate.json`；缺失或欄位不完整時，agent 會向使用者詢問四個欄位並寫入。

| 欄位 | 必填 | 說明 |
|------|------|------|
| `author_name` | 是 | 作者姓名，顯示於版權頁尾與 LICENSE |
| `author_email` | 是 | 聯絡 Email（Proprietary LICENSE 使用） |
| `author_url` | 是 | 個人連結（LinkedIn / GitHub / 個人網站） |
| `github_owner` | 是 | GitHub 使用者名稱，用於預設 `{owner}` |

範例 `~/.skill-readme-generate.json`：

```json
{
  "author_name": "張三 John Doe",
  "author_email": "dev@example.com",
  "author_url": "https://linkedin.com/in/johndoe",
  "github_owner": "johndoe"
}
```

### 手動初始化

於終端機直接執行腳本可在 harness 外建立或檢視設定：

```bash
python3 <skills-dir>/readme-generate/scripts/setup_config.py
```

若檔案已存在則印出現有設定；若不存在則以 `input()` 逐欄詢問。stdin 非 TTY 時以 exit 2 結束。

### 非互動寫入

```bash
python3 <skills-dir>/readme-generate/scripts/setup_config.py write \
    "張三 John Doe" \
    "dev@example.com" \
    "https://linkedin.com/in/johndoe" \
    "johndoe"
```

### 覆蓋機制

指令列傳入 `REPO_PATH`（含 `github.com/{owner}/{repo}`）時，`{owner}` 取自該路徑，其餘作者欄位仍由設定檔提供。直接編輯或刪除 `~/.skill-readme-generate.json` 即可更新或觸發重新設定。

## 使用方式

### 基本用法

```bash
/readme-generate
```

於 harness 當前工作目錄執行：

1. 載入或建立作者設定
2. 執行 `analyze_project.py` 分析專案
3. 提煉 3–5 個特色，生成六個雙語文件
4. 若無 LICENSE 則預設產生 MIT

### 指定授權類型

```bash
/readme-generate Apache-2.0
```

### 私有模式

```bash
/readme-generate private
```

README 跳過封面、標語、徽章、授權與作者區段，版權頁尾僅保留 `©️ {year}`。

### 覆蓋儲存庫路徑

```bash
/readme-generate github.com/foo/bar
```

將所有 GitHub URL 的 `{owner}/{repo}` 替換為 `foo/bar`。

### 組合使用

```bash
/readme-generate private MIT github.com/foo/bar
```

參數順序無關。

### 手動執行原始碼分析

```bash
python3 <skills-dir>/readme-generate/scripts/analyze_project.py /path/to/project
```

輸出包含語言、名稱、版本、檔案清單、匯出型別、函式與相依性的 JSON，可用於除錯或整合至其他工具。

## 命令列參考

### Slash Command 參數

| 參數 | 格式 | 說明 |
|------|------|------|
| `private` | 關鍵字（不區分大小寫） | 跳過封面、標語、徽章、授權與作者區段 |
| `LICENSE_TYPE` | 授權識別碼 | 生成對應的 LICENSE 檔案 |
| `REPO_PATH` | `github.com/{owner}/{repo}` | 覆蓋自動偵測的擁有者與儲存庫 |

### 支援的授權類型

| 類型 | 別名（不區分大小寫） |
|------|----------------------|
| MIT | `mit` |
| Apache-2.0 | `apache`、`apache2`、`apache-2.0` |
| GPL-3.0 | `gpl`、`gpl3`、`gpl-3.0` |
| BSD-3-Clause | `bsd`、`bsd3`、`bsd-3-clause` |
| ISC | `isc` |
| Unlicense | `unlicense`、`public-domain` |
| Proprietary | `proprietary`（自動啟用 `private` 模式） |

### 輸出檔案

| 檔案 | 說明 |
|------|------|
| `README.md` | 英文主要文件，置於專案根目錄 |
| `doc/README.zh.md` | 繁體中文版本 |
| `doc/doc.md` | 英文詳細技術文件 |
| `doc/doc.zh.md` | 繁體中文詳細技術文件 |
| `doc/architecture.md` | 英文詳細架構圖 |
| `doc/architecture.zh.md` | 繁體中文詳細架構圖 |
| `LICENSE` | 依指定類型生成；未指定且不存在時預設 MIT |

### setup_config.py 子指令

| 指令 | stdout | exit |
|------|--------|------|
| `setup_config.py` | 設定 JSON（缺失時先互動詢問） | `0`；非 TTY 為 `2` |
| `setup_config.py check` | 設定 JSON；缺失時 stdout 為空、stderr 印 `MISSING` | `0` / `1` |
| `setup_config.py write NAME EMAIL URL OWNER` | 寫入後的設定 JSON | `0`；參數數量錯誤或空值為 `2` |

### analyze_project.py

| 參數 | 說明 |
|------|------|
| `<project_path>` | 要分析的專案根目錄；缺少參數時 exit 1，路徑不存在時輸出 `{"error": ...}` |

語言偵測先比對指標檔（`go.mod`、`pyproject.toml`、`package.json`、`tsconfig.json`、`composer.json`、`Package.swift`），無命中再依副檔名數量決定。完整解析：Python（AST）、Go、JavaScript、TypeScript；其他語言僅輸出檔案清單。輸出 JSON 欄位：`language`、`name`、`description`、`version`、`files`、`types`、`functions`、`dependencies`。

### 參數優先順序

`{owner}` 與 `{repo}` 解析順序：

1. 指令列 `REPO_PATH`（最高優先）
2. `~/.skill-readme-generate.json` 的 `github_owner`
3. 本地 `git remote get-url origin`
4. 當前資料夾名稱（最低優先）
