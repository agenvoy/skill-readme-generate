# readme-generate - 技術文件

> 返回 [README](./README.zh.md)

## 前置需求

- Python 3.10 或更高版本（腳本使用 `dict | None` 聯集型別語法）
- [Claude Code](https://claude.ai/claude-code) CLI 已安裝並設定完成
- Git（用於讀取 `git remote`、首次提交年份等資訊）

## 安裝

### 從 GitHub 複製

```bash
git clone https://github.com/agenvoy/skill-readme-generate.git \
    ~/.claude/skills/readme-generate
```

### 手動安裝

將下列檔案放置於 `~/.claude/skills/readme-generate/`：

```
readme-generate/
├── SKILL.md                  # Skill 定義與流程協議
├── scripts/
│   ├── analyze_project.py    # 原始碼分析腳本
│   ├── setup_config.py       # 作者設定腳本
│   ├── examples/             # README / doc 生成藍本
│   └── licenses/             # 開源授權範本
├── LICENSE
├── README.md
└── doc/
    ├── README.zh.md
    ├── doc.md
    ├── doc.zh.md
    ├── architecture.md
    └── architecture.zh.md
```

安裝完成後，於 Claude Code 中以 `/readme-generate` 呼叫即可。

## 設定

### 作者設定檔

每次執行 `/readme-generate` 都會先以 `setup_config.py check` 檢查 `~/.skill-readme-generate.json`；缺失或欄位不完整時，Claude 會以 `AskUserQuestion` 詢問四個欄位並寫入。

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

於終端機直接執行腳本可在 Claude Code 外建立或檢視設定：

```bash
python3 ~/.claude/skills/readme-generate/scripts/setup_config.py
```

若檔案已存在則印出現有設定；若不存在則以 `input()` 逐欄詢問。stdin 非 TTY 時以 exit 2 結束。

### 非互動寫入

```bash
python3 ~/.claude/skills/readme-generate/scripts/setup_config.py write \
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

於當前 Claude Code 工作目錄執行：

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

### 只重生成部分檔案

```bash
/readme-generate --only readme
/readme-generate --only doc,architecture
```

僅覆寫指定目標對應的檔案集；未指定的文件與 LICENSE 不讀取、不覆寫，且忽略 `LICENSE_TYPE`。

### usage 教學模式

```bash
/readme-generate usage
```

`README.md` 與 `doc/README.zh.md` 整份改寫為純使用說明（前置需求 / 安裝 / 設定 / 使用方式 / 參考），不觸碰 `doc/doc.md`、`doc/architecture.md` 與 LICENSE。

### 覆蓋儲存庫路徑

```bash
/readme-generate github.com/foo/bar
```

將所有 GitHub URL 的 `{owner}/{repo}` 替換為 `foo/bar`。

### 組合使用

```bash
/readme-generate private MIT github.com/foo/bar
/readme-generate private --only readme
```

位置參數順序無關；`--only` 與其值視為一組。

### 手動執行原始碼分析

```bash
python3 ~/.claude/skills/readme-generate/scripts/analyze_project.py /path/to/project
```

輸出包含語言、名稱、版本、檔案清單、匯出型別、函式與相依性的 JSON，可用於除錯或整合至其他工具。

## 命令列參考

### Slash Command 參數

| 參數 | 格式 | 說明 |
|------|------|------|
| `private` | 關鍵字（不區分大小寫） | 跳過封面、標語、徽章、授權與作者區段 |
| `usage` | 關鍵字（不區分大小寫） | 僅生成純使用說明版 README 兩檔；強制目標集為 `readme`，忽略 `LICENSE_TYPE` |
| `LICENSE_TYPE` | 授權識別碼 | 生成對應的 LICENSE 檔案（`--only` 或 `usage` 時忽略） |
| `REPO_PATH` | `github.com/{owner}/{repo}` | 覆蓋自動偵測的擁有者與儲存庫 |
| `--only <targets>` | 逗號分隔，亦接受 `--only=<targets>` | 僅重生成指定目標 |

### `--only` 目標

| Target（不區分大小寫） | 重新生成檔案 |
|------|------|
| `readme` | `README.md` + `doc/README.zh.md` |
| `doc` | `doc/doc.md` + `doc/doc.zh.md` |
| `architecture` | `doc/architecture.md` + `doc/architecture.zh.md` |

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
| `LICENSE` | 無 `--only` 且非 `usage` 時處理；未指定類型且不存在時預設 MIT |

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
