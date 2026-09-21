# readme-generate - Documentation

> Back to [README](../README.md)

## Prerequisites

- Python 3.10 or higher (the scripts use `dict | None` union syntax)
- An agent harness that loads `SKILL.md` skills and can run shell commands
- Git (used to read `git remote` and the first commit year)

## Installation

`<skills-dir>` is the skill directory your harness scans.

### Clone from GitHub

```bash
git clone https://github.com/agenvoy/skill-readme-generate.git \
    <skills-dir>/readme-generate
```

### Manual Installation

Place the following files under `<skills-dir>/readme-generate/`:

```
readme-generate/
├── SKILL.md                  # Skill definition and protocol
├── LICENSE
└── scripts/
    ├── analyze_project.py    # Source analysis script
    ├── setup_config.py       # Author config script
    ├── examples/             # README / doc generation blueprints
    └── licenses/             # Open-source license templates
```

Invoke it from your harness with `/readme-generate`.

## Configuration

### Author Config File

Every `/readme-generate` run first checks `~/.skill-readme-generate.json` via `setup_config.py check`; when the file is missing or incomplete, the agent asks the user for the four fields and writes them.

| Field | Required | Description |
|-------|----------|-------------|
| `author_name` | Yes | Author name, shown in the copyright footer and LICENSE |
| `author_email` | Yes | Contact email (used by the Proprietary LICENSE) |
| `author_url` | Yes | Personal URL (LinkedIn / GitHub / homepage) |
| `github_owner` | Yes | GitHub username, used as the default `{owner}` |

Example `~/.skill-readme-generate.json`:

```json
{
  "author_name": "張三 John Doe",
  "author_email": "dev@example.com",
  "author_url": "https://linkedin.com/in/johndoe",
  "github_owner": "johndoe"
}
```

### Manual Initialization

Run the script in a terminal to create or view the config outside the harness:

```bash
python3 <skills-dir>/readme-generate/scripts/setup_config.py
```

It prints the existing config if present; otherwise it prompts for each field with `input()`. It exits with code 2 when stdin is not a TTY.

### Non-Interactive Write

```bash
python3 <skills-dir>/readme-generate/scripts/setup_config.py write \
    "張三 John Doe" \
    "dev@example.com" \
    "https://linkedin.com/in/johndoe" \
    "johndoe"
```

### Overrides

When `REPO_PATH` (`github.com/{owner}/{repo}`) is passed on the command line, `{owner}` comes from that path while other author fields still come from the config file. Edit or delete `~/.skill-readme-generate.json` to update it or trigger a reset.

## Usage

### Basic

```bash
/readme-generate
```

Runs in the harness's current working directory:

1. Load or create the author config
2. Run `analyze_project.py` on the project
3. Extract 3–5 features and generate the six bilingual docs
4. Generate an MIT LICENSE if none exists

### Choose a License

```bash
/readme-generate Apache-2.0
```

### Private Mode

```bash
/readme-generate private
```

The README skips the cover, tagline, badges, license, and author sections; the copyright footer keeps only `©️ {year}`.

### Override Repository Path

```bash
/readme-generate github.com/foo/bar
```

Replaces `{owner}/{repo}` in every GitHub URL with `foo/bar`.

### Combined

```bash
/readme-generate private MIT github.com/foo/bar
```

Arguments are order-independent.

### Run Source Analysis Manually

```bash
python3 <skills-dir>/readme-generate/scripts/analyze_project.py /path/to/project
```

Outputs JSON with language, name, version, file list, exported types, functions, and dependencies, useful for debugging or integration with other tools.

## CLI Reference

### Slash Command Arguments

| Argument | Format | Description |
|----------|--------|-------------|
| `private` | Keyword (case-insensitive) | Skip cover, tagline, badges, license, and author sections |
| `LICENSE_TYPE` | License identifier | Generate the matching LICENSE file |
| `REPO_PATH` | `github.com/{owner}/{repo}` | Override the detected owner and repository |

### Supported Licenses

| Type | Aliases (case-insensitive) |
|------|----------------------------|
| MIT | `mit` |
| Apache-2.0 | `apache`, `apache2`, `apache-2.0` |
| GPL-3.0 | `gpl`, `gpl3`, `gpl-3.0` |
| BSD-3-Clause | `bsd`, `bsd3`, `bsd-3-clause` |
| ISC | `isc` |
| Unlicense | `unlicense`, `public-domain` |
| Proprietary | `proprietary` (implies `private` mode) |

### Output Files

| File | Description |
|------|-------------|
| `README.md` | English main document in the project root |
| `doc/README.zh.md` | Traditional Chinese version |
| `doc/doc.md` | English detailed technical documentation |
| `doc/doc.zh.md` | Traditional Chinese detailed technical documentation |
| `doc/architecture.md` | English detailed architecture diagrams |
| `doc/architecture.zh.md` | Traditional Chinese detailed architecture diagrams |
| `LICENSE` | Generated for the given type; defaults to MIT when unspecified and absent |

### setup_config.py Subcommands

| Command | stdout | Exit |
|---------|--------|------|
| `setup_config.py` | Config JSON (prompts first when missing) | `0`; `2` when not a TTY |
| `setup_config.py check` | Config JSON; empty stdout and `MISSING` on stderr when missing | `0` / `1` |
| `setup_config.py write NAME EMAIL URL OWNER` | The written config JSON | `0`; `2` on wrong argument count or empty values |

### analyze_project.py

| Argument | Description |
|----------|-------------|
| `<project_path>` | Project root to analyze; exits 1 when omitted, outputs `{"error": ...}` when the path does not exist |

Language detection first matches indicator files (`go.mod`, `pyproject.toml`, `package.json`, `tsconfig.json`, `composer.json`, `Package.swift`), then falls back to the most frequent source extension. Full parsing: Python (AST), Go, JavaScript, TypeScript; other languages output only the file list. Output JSON fields: `language`, `name`, `description`, `version`, `files`, `types`, `functions`, `dependencies`.

### Resolution Order

`{owner}` and `{repo}` are resolved in this order:

1. `REPO_PATH` on the command line (highest)
2. `github_owner` from `~/.skill-readme-generate.json`
3. Local `git remote get-url origin`
4. Current folder name (lowest)
