# Changelog

完整規範以 `SKILL.md` 與 `scripts/` 為準（＝最新規範）；本檔只用於快速定位專案既有 `README.md`、`doc/README.zh.md`、`doc/doc*.md`、`doc/architecture*.md` 與最新規範的差異，命中即直接修改專案。

最新改動：2026-10-06

## 破壞性變更

- Coverage 徽章移除：刪除徽章列中 `alt="Coverage"`（`app.codecov.io`／`img.shields.io/codecov`）那一行
- 輸出檔開頭新增最後更新日期：`README.md`／`doc/README.zh.md` 第一行加 `Last updated: {updated}`／`最後更新：{updated}`；`doc/doc*.md`、`doc/architecture*.md` 於 `# 標題` 下一行加同格式日期
- 順序 3 簡短描述不再自行生成：專案 `.doc/seo-optimize/config.json`（或 `wiki-worker/.doc/seo-optimize/config.json`）有 `one_liner` → README.md／中文 README 的順序 3 blockquote 改為逐字 `one_liner.en`／`one_liner.zh`；無則依順序 3 狀態表處理
- `usage` 模式移除：以 `usage` 產出的 README（無功能特點、主體為前置需求／安裝／使用方式／參考）→ 依現行區段順序重寫，並補齊 `doc/doc*.md`、`doc/architecture*.md`、LICENSE
- Author 區段改為 `Just [open an issue](https://github.com/{owner}/{repo}/issues/new) to share an idea.` ＋ contrib.rocks 貢獻者圖（`cache_bust={date}`）；舊格式（`github.com/{owner}.png` 或 `avatars.githubusercontent.com` 頭像、`<h4>` 姓名、email／個人連結或圖示）整段替換
- 通用徽章連結互換錯誤：Version 徽章 `href="LICENSE"` → `https://github.com/{owner}/{repo}/releases`；License 徽章 `href=".../releases"` → `LICENSE`
- `doc/README.zh.md` 功能特點 blockquote 的 `[完整文件](./doc/README.zh.md)` → `[完整文件](./doc.zh.md)`（舊連結指向自己）
- LLM 生成通知連結 `github.com/pardnchiu/skill-readme-generate` → `github.com/agenvoy/skill-readme-generate`
- Stars 區段移除：刪 `## Stars`（star-history 圖）與目錄 `[Stars](#stars)`
- 技術堆疊區段移除：刪 `## 技術堆疊`／`## Built With`（skillicons 圖）與對應目錄項
- 私有模式不再輸出封面、標語區塊（含其後 `***`）、授權、Author；版權頁尾去掉作者連結僅留 `©️ {year}`
- 徽章集調整：Go Application（僅 `main` package）刪 Go Reference 與 Coverage；Go／Node.js／PHP 不再附加通用 Version／License 徽章（改用各自徽章集）；Node.js Downloads `npm/dm` → `jsdelivr/npm/hm`
- 檔案結構區段移除：刪 `## 檔案結構`／`## File Structure` 與對應目錄項
- 功能特點由 `### 特色標題` 子區段改為 3–5 項 list（`- **標題** — 一句話說明`）
- 標題區改版：`# {repo}` ＋ markdown 徽章 → 置中 `<strong>` 英文大寫標語 ＋ 置中 HTML `for-the-badge` 徽章 ＋ `***`；刪 `pkg.go.dev/badge`、`goreportcard.com` 等非 shields.io 徽章
- 輸出路徑移至 `doc/`：根目錄的 `README.zh.md`、`doc.md`、`doc.zh.md` → 移入 `doc/` 並刪除根目錄舊檔，相對連結一併改（`../README.md`、`./doc/doc.md`、`./doc/README.zh.md`）
- README 不再含 Installation／Usage／Reference／使用案例區段：移至 `doc/doc*.md`，README 刪除這些區段與目錄項
