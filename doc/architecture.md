# readme-generate - Architecture

> Back to [README](../README.md)

## Overview

```mermaid
graph TB
    User[User] -->|/readme-generate| SKILL[SKILL.md<br/>Orchestration]
    SKILL --> Parser[Argument Parser<br/>private / usage / LICENSE_TYPE<br/>REPO_PATH / --only]
    SKILL --> Config[setup_config.py<br/>Author Config]
    SKILL --> Analyze[analyze_project.py<br/>Source Analysis]
    Config --> JSON[~/.skill-readme-generate.json]
    Parser --> Target[Target Set<br/>readme / doc / architecture]
    Analyze --> Data[Language / Types / Functions<br/>Dependencies / Files]
    Examples[scripts/examples<br/>Blueprints] --> Generator
    Licenses[scripts/licenses<br/>License Templates] --> Generator
    Target --> Generator[Generator<br/>Chinese First → English]
    Data --> Generator
    JSON --> Generator
    Generator --> Output[README.md / doc/README.zh.md<br/>doc/doc.md / doc.zh.md<br/>doc/architecture.md / architecture.zh.md<br/>LICENSE]
```

## Module: SKILL.md (Orchestration)

Defines the workflow, section ordering, and validation checklist Claude follows when executing the skill. Contains no executable code; it constrains LLM behavior through prompt instructions.

```mermaid
graph TB
    subgraph SKILL["SKILL.md"]
        Step0[0 Author Config] --> Step1[1 Parse Arguments<br/>Resolve Target Set]
        Step1 --> Step2[2 Analyze Project]
        Step2 --> Step3[3 Extract Params<br/>owner / repo / year]
        Step3 --> Step4[4 Review Existing Docs]
        Step4 --> Step5[5 Extract 3–5 Features<br/>skipped in usage]
        Step5 --> Step6[6 Generate readme]
        Step6 --> Step7[7 Generate doc]
        Step7 --> Step8[8 Generate architecture]
        Step8 --> Step9[9 LICENSE<br/>full run, non-usage only]
        Step9 --> Step10[10 Validation Checklist]
    end
    SlashCmd[/readme-generate/] --> SKILL
    SKILL --> Files[Target Files + LICENSE]
```

Each generation step runs only when its target is in the target set; files outside the set are neither read nor overwritten.

## Module: setup_config.py (Author Config)

Provides interactive creation, non-interactive write, and existence check. The config is stored as UTF-8 JSON at `~/.skill-readme-generate.json`, and all four fields must be non-empty strings.

```mermaid
graph TB
    subgraph Config["setup_config.py"]
        Main[main<br/>Subcommand Dispatch] --> Check[cmd_check<br/>Load and Validate]
        Main --> Write[cmd_write<br/>Four-Arg Write]
        Main --> Default[cmd_default<br/>Prompt or Print]
        Check --> Load[load_config<br/>Read + Field Validation]
        Default --> Load
        Default --> Prompt[prompt_interactive<br/>TTY input]
        Prompt --> Save[write_config<br/>UTF-8 JSON]
        Write --> Save
    end
    JSON[~/.skill-readme-generate.json] <--> Load
    JSON <--> Save
    SKILL[SKILL.md Step 0] -->|check / write| Main
```

**Input / Output**:

| Subcommand | stdin | stdout | stderr | Exit |
|------------|-------|--------|--------|------|
| `check` | - | JSON | `MISSING` when absent | 0 / 1 |
| `write` | - | JSON | Save path or error | 0 / 2 |
| default | TTY | JSON | Prompts | 0 / 2 |

## Module: analyze_project.py (Source Analysis)

Detects the primary language, dispatches to the matching extractor, and serializes a unified `ProjectAnalysis` to JSON.

```mermaid
graph TB
    subgraph Analyzer["analyze_project.py"]
        Entry[analyze_project<br/>Entry] --> Detect[detect_language]
        Detect --> Indicators[_detect_by_indicators<br/>Indicator Files]
        Detect --> Ext[_detect_by_extensions<br/>Extension Count]
        Detect -->|go| Go[extract_go_info<br/>go.mod / type / func]
        Detect -->|python| Py[extract_python_info<br/>pyproject.toml / AST]
        Detect -->|javascript / typescript| JS[extract_js_ts_info<br/>package.json / export]
        Detect -->|other| Fallback[_list_generic_files<br/>File List Only]
        Go --> Result[ProjectAnalysis]
        Py --> Result
        JS --> Result
        Fallback --> Result
        Result --> Serialize[asdict + json.dumps]
    end
    SKILL[SKILL.md Step 2] -->|project_path| Entry
    Serialize -->|stdout JSON| SKILL
```

**Data Classes**:

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

`entry_points` is a data class field but is not included in the output JSON.

## Data Flow

Full flow of a single `/readme-generate` invocation:

```mermaid
sequenceDiagram
    participant User
    participant Claude as Claude Code
    participant Skill as SKILL.md
    participant Config as setup_config.py
    participant Analyze as analyze_project.py
    participant FS as File System

    User->>Claude: /readme-generate [args]
    Claude->>Skill: Load skill definition
    Skill->>Config: setup_config.py check
    alt Config missing
        Config-->>Skill: exit 1
        Skill->>User: AskUserQuestion (4 fields)
        User-->>Skill: Author info
        Skill->>Config: setup_config.py write ...
        Config->>FS: Write ~/.skill-readme-generate.json
        Config-->>Skill: JSON
    else Config complete
        Config-->>Skill: JSON
    end
    Skill->>Skill: Parse arguments and resolve target set
    Skill->>Analyze: analyze_project.py <path>
    Analyze->>FS: Recursively scan sources
    Analyze-->>Skill: ProjectAnalysis JSON
    opt readme ∈ target set
        Skill->>FS: Write doc/README.zh.md → README.md
    end
    opt doc ∈ target set
        Skill->>FS: Write doc/doc.zh.md → doc/doc.md
    end
    opt architecture ∈ target set
        Skill->>FS: Write doc/architecture.zh.md → doc/architecture.md
    end
    opt No --only, not usage, and (type given or no LICENSE)
        Skill->>FS: Write LICENSE
    end
    Skill-->>User: Completion notice
```

## Argument Parsing State Machine

Argument detection and target set resolution:

```mermaid
stateDiagram-v2
    [*] --> Token: Read next token
    Token --> OnlyCheck: Token present
    Token --> Resolve: No token
    OnlyCheck --> SetOnly: --only or --only=
    OnlyCheck --> PrivateCheck: No match
    PrivateCheck --> SetPrivate: private
    PrivateCheck --> UsageCheck: No match
    UsageCheck --> SetUsage: usage
    UsageCheck --> RepoCheck: No match
    RepoCheck --> SetRepo: Contains github.com/
    RepoCheck --> LicenseCheck: No match
    LicenseCheck --> SetLicense: Known license alias
    LicenseCheck --> Ignore: No match
    SetOnly --> Token
    SetPrivate --> Token
    SetUsage --> Token
    SetRepo --> Token
    SetLicense --> Token
    Ignore --> Token
    Resolve --> UsageTarget: USAGE_MODE
    Resolve --> OnlyTarget: ONLY_TARGETS non-empty
    Resolve --> FullTarget: Otherwise
    UsageTarget --> Finalize: target = readme, ignore LICENSE_TYPE
    OnlyTarget --> Finalize: target = given, ignore LICENSE_TYPE
    FullTarget --> Finalize: target = all + LICENSE
    Finalize --> [*]: proprietary implies private
```
