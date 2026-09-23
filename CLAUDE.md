# 專案規則

## 文件學習流程（docs/llm）

使用者要求「下載某頁 TypeSafe 文件並做中文版」時，依下列流程處理：

1. **下載原文**：從 `https://docs.typesafe.ai/<路徑>.md` 用 `curl -sSfL` 下載，內容不做任何修改。
   - 存到 `docs/llm/`，保留線上的路徑結構。例如 `/introduction.md` → `docs/llm/introduction.md`；`/primitives/choice.md` → `docs/llm/primitives/choice.md`。
2. **建立繁體中文版**：在同一個資料夾建立 `<檔名>.zh.md`，例如 `docs/llm/introduction.zh.md`。
   - 檔案開頭加上引用區塊，註明原文檔名（相對連結）、來源 URL、下載日期，並說明內容有出入時以線上英文版為準。
   - 完整翻譯全文，不要摘要，也不要省略段落。
   - 站內相對連結（例如 `/concepts/system-one`）改成絕對網址並加上 `.md`，例如 `https://docs.typesafe.ai/concepts/system-one.md`。若該頁已經下載到 `docs/llm/`，改連到本地的 `.zh.md`。
   - 程式碼區塊（含 JSON、提示詞範例）內容一律保持原樣，不翻譯；若其中英文字串對理解有幫助（例如範例狀態、instructions、criteria、提示詞），在區塊後加上「> 中文對照：…」引用區塊。
   - Mintlify 程式碼區塊的屬性（`theme={null}`、`title="…"`、`wrap`）移除；有 `title` 時改成區塊上方的粗體標題。
   - `<Tabs>`／`<Tab>` 改成「**方式一：…**」這類粗體小標題依序列出。
   - 頁面中 `export function …` 的網站元件程式碼（例如 `TypesafeExample`）省略，並在開頭聲明中註明；`<TypesafeExample example={{…}} />` 改寫成等價的 JSON 程式碼區塊，並附上 Playground 連結。
   - `<Columns>`／`<Card>` 改成條列連結。
   - 其他互動元件（例如 `<ConfidenceExplorer />`）從元件程式碼中取出給人看的文字（標題、按鈕、說明），改寫成文字或表格說明；譯者自行補充的計算或解讀以「（譯註：…）」標明。
   - 頁內錨點（例如 `#ask-multiple-questions-together`）改成指向中文標題的錨點（例如 `#一起提出多個問題`）。
   - 新頁完成後，檢查其他已翻譯的 `.zh.md` 是否有連到這一頁，有的話一併改成本地連結。Mermaid 圖可以翻譯節點文字；把 Mintlify 專用的屬性（例如 `actions={true} theme={null}`）移除，讓 GitHub 能正常渲染。
   - Mintlify 專用元件（例如 `<Note>`、`<Tip>`、`<Card>`）轉換成一般 Markdown，例如用引用區塊表示。
3. **更新索引**：在 [docs/llms.zh.md](docs/llms.zh.md) 中，把對應條目的連結改成指向本地的 `.zh.md`，並保留線上原文連結（例如在條目後加上「（[原文](…)）」）。

### 翻譯用詞

與 [README.zh.md](README.zh.md)、[docs/SKILL.zh.md](docs/SKILL.zh.md)、[docs/llms.zh.md](docs/llms.zh.md) 保持一致：

| 英文 | 中文 |
| --- | --- |
| primitive | 原語 |
| state | 狀態 |
| question | 問題 |
| instructions / criteria | 指示 / 準則 |
| confidence | 信心度 |
| probability | 機率 |
| cookbook | 操作手冊 |
| pattern | 模式 |
| agent | Agent |
| typed | 具型別的 |
| speculative fan-out | 推測式扇出 |
| calibrated | 經過校準的 |

- Choice、Score、Noul、System One、Jev 等專有名詞，以及 SDK 的類別、函式、欄位名稱，都保留英文。
- 重要術語第一次出現時，可以在括號中附上英文原文。
- 內文中以行內程式碼出現、本身是英文單字的選項值或識別名稱（例如 `refund`、`other`），保留原文並在後面加上中文括註，例如 `refund`（退款）。欄位名稱（例如 `confidence`、`probabilities`）若前後文已說明含義則不必再括註。
