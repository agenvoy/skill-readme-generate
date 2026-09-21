# readme-generate - 架構

> 返回 [README](./README.zh.md)

## 概覽

```mermaid
graph TB
    User[使用者] -->|/readme-generate| SKILL[SKILL.md<br/>流程協調]
    SKILL --> Parser[參數解析<br/>private / LICENSE_TYPE / REPO_PATH]
    SKILL --> Config[setup_config.py<br/>作者設定]
    SKILL --> Analyze[analyze_project.py<br/>原始碼分析]
    Config --> JSON[~/.skill-readme-generate.json]
    Analyze --> Data[語言 / 型別 / 函式<br/>相依 / 檔案清單]
    Examples[scripts/examples<br/>生成藍本] --> Generator
    Licenses[scripts/licenses<br/>授權範本] --> Generator
    Parser --> Generator[生成器<br/>中文先行 → 英文翻譯]
    Data --> Generator
    JSON --> Generator
    Generator --> Output[README.md / doc/README.zh.md<br/>doc/doc.md / doc.zh.md<br/>doc/architecture.md / architecture.zh.md<br/>LICENSE]
```

## Module: SKILL.md（流程協調）

定義 agent 執行此 skill 時需遵守的工作流程、區段順序與驗證清單。本身不含執行碼，僅以提示詞約束 LLM 行為；腳本路徑以 `{skill_dir}` 表示，依實際載入位置代入。

```mermaid
graph TB
    subgraph SKILL["SKILL.md"]
        Step0[0 作者設定] --> Step1[1 解析參數]
        Step1 --> Step2[2 分析專案]
        Step2 --> Step3[3 提取參數<br/>owner / repo / year]
        Step3 --> Step4[4 檢視既有文件]
        Step4 --> Step5[5 提煉 3–5 特色]
        Step5 --> Step6[6 生成 readme]
        Step6 --> Step7[7 生成 doc]
        Step7 --> Step8[8 生成 architecture]
        Step8 --> Step9[9 LICENSE]
        Step9 --> Step10[10 驗證清單]
        Step10 --> Step11[11 儲存]
    end
    SlashCmd[/readme-generate/] --> SKILL
    SKILL --> Files[六檔輸出 + LICENSE]
```

## Module: setup_config.py（作者設定）

提供互動建立、非互動寫入、存在性檢查三種模式。設定以 UTF-8 JSON 存放於 `~/.skill-readme-generate.json`，四個欄位皆須為非空字串。

```mermaid
graph TB
    subgraph Config["setup_config.py"]
        Main[main<br/>子指令分派] --> Check[cmd_check<br/>載入並驗證]
        Main --> Write[cmd_write<br/>四參數寫入]
        Main --> Default[cmd_default<br/>互動或印出]
        Check --> Load[load_config<br/>讀取 + 欄位驗證]
        Default --> Load
        Default --> Prompt[prompt_interactive<br/>TTY input]
        Prompt --> Save[write_config<br/>UTF-8 JSON]
        Write --> Save
    end
    JSON[~/.skill-readme-generate.json] <--> Load
    JSON <--> Save
    SKILL[SKILL.md Step 0] -->|check / write| Main
```

**輸入／輸出**：

| 子指令 | stdin | stdout | stderr | exit |
|--------|-------|--------|--------|------|
| `check` | - | JSON | 缺失時 `MISSING` | 0 / 1 |
| `write` | - | JSON | 儲存路徑或錯誤 | 0 / 2 |
| 預設 | TTY | JSON | 提示文字 | 0 / 2 |

## Module: analyze_project.py（原始碼分析）

偵測主要語言後調用對應 extractor 提取結構資訊，輸出統一的 `ProjectAnalysis` 序列化 JSON。

```mermaid
graph TB
    subgraph Analyzer["analyze_project.py"]
        Entry[analyze_project<br/>入口] --> Detect[detect_language]
        Detect --> Indicators[_detect_by_indicators<br/>指標檔]
        Detect --> Ext[_detect_by_extensions<br/>副檔名計數]
        Detect -->|go| Go[extract_go_info<br/>go.mod / type / func]
        Detect -->|python| Py[extract_python_info<br/>pyproject.toml / AST]
        Detect -->|javascript / typescript| JS[extract_js_ts_info<br/>package.json / export]
        Detect -->|其他| Fallback[_list_generic_files<br/>僅檔案清單]
        Go --> Result[ProjectAnalysis]
        Py --> Result
        JS --> Result
        Fallback --> Result
        Result --> Serialize[asdict + json.dumps]
    end
    SKILL[SKILL.md Step 2] -->|project_path| Entry
    Serialize -->|stdout JSON| SKILL
```

**資料類別**：

```mermaid
classDiagram
    class ProjectAnalysis {
        +str language
        +str name
        +str description
        +str version
        +list~TypeInfo~ types
        +list~FunctionInfo~ functions
        +list~str~ files
        +list~str~ dependencies
        +list~str~ entry_points
    }
    class TypeInfo {
        +str name
        +str kind
        +list~dict~ fields
        +str doc
        +str file
    }
    class FunctionInfo {
        +str name
        +str signature
        +str doc
        +bool exported
        +str file
        +int line
    }
    ProjectAnalysis --> TypeInfo
    ProjectAnalysis --> FunctionInfo
```

`entry_points` 為資料類別欄位，但不包含在輸出 JSON 中。

## 資料流

單次 `/readme-generate` 呼叫的完整流程：

```mermaid
sequenceDiagram
    participant User as 使用者
    participant Agent as Agent Harness
    participant Skill as SKILL.md
    participant Config as setup_config.py
    participant Analyze as analyze_project.py
    participant FS as 檔案系統

    User->>Agent: /readme-generate [args]
    Agent->>Skill: 載入 skill 定義
    Skill->>Config: setup_config.py check
    alt 設定缺失
        Config-->>Skill: exit 1
        Skill->>User: 詢問四欄位
        User-->>Skill: 作者資訊
        Skill->>Config: setup_config.py write ...
        Config->>FS: 寫入 ~/.skill-readme-generate.json
        Config-->>Skill: JSON
    else 設定完整
        Config-->>Skill: JSON
    end
    Skill->>Skill: 解析 PRIVATE_MODE / LICENSE_TYPE / REPO_PATH
    Skill->>Analyze: analyze_project.py <path>
    Analyze->>FS: 遞迴掃描原始檔
    Analyze-->>Skill: ProjectAnalysis JSON
    Skill->>FS: 寫入 doc/README.zh.md → README.md
    Skill->>FS: 寫入 doc/doc.zh.md → doc/doc.md
    Skill->>FS: 寫入 doc/architecture.zh.md → doc/architecture.md
    opt 指定 LICENSE_TYPE 或無 LICENSE
        Skill->>FS: 寫入 LICENSE
    end
    Skill-->>User: 完成通知
```

## 參數解析狀態機

三個選填參數的偵測與分類：

```mermaid
stateDiagram-v2
    [*] --> Token: 讀取下一個 token
    Token --> PrivateCheck: token 存在
    Token --> Finalize: 無 token
    PrivateCheck --> SetPrivate: private
    PrivateCheck --> RepoCheck: 不符
    RepoCheck --> SetRepo: 含 github.com/
    RepoCheck --> LicenseCheck: 不符
    LicenseCheck --> SetLicense: 符合授權別名
    LicenseCheck --> Ignore: 全部不符
    SetPrivate --> Token
    SetRepo --> Token
    SetLicense --> Token
    Ignore --> Token
    Finalize --> [*]: proprietary 隱含 private
```
