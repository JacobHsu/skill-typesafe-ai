# TypeSafe AI 技能（繁體中文版）

> 本文件是 [`skills/typesafe-ai/SKILL.md`](../skills/typesafe-ai/SKILL.md) 的繁體中文翻譯，供人類閱讀參考。
> Agent 實際載入的是英文原檔；若兩者內容有出入，以英文原檔為準。

## 技能資訊

| 欄位 | 內容 |
| --- | --- |
| 名稱 | `typesafe-ai` |
| 授權 | MIT |

**描述：** 使用 TypeSafe 建置 AI 驅動的軟體：TypeSafe 提供小型的 AI 智慧單元，讓你能像使用程式設計原語（primitive）一樣使用它們。其 System One 模型（包括 Jev）能將自然語言與應用程式狀態轉換為具型別的判斷與機率，供程式碼組合運用。適用時機：某項功能需要「可程式化的常識」、在發想 AI 能為應用程式帶來哪些可能性，或某個「LLM 提示後再解析輸出」的步驟可以改成結構化決策時。應用範圍包括路由分流、排序、擷取、驗證與互動式體驗；這些只是起點，而非極限。請閱讀線上文件與操作手冊（cookbook），找出有用的模式並發掘新的組合方式。

---

# 使用 TypeSafe 建置

TypeSafe 讓 AI 智慧單元能像程式設計原語一樣使用：一個個小型判斷，可以組合成更大的能力。其 **System One 模型** 回傳快速、聚焦的判斷，軟體可以直接取用。**Jev** 是 TypeSafe 的旗艦模型，也是第一個 System One 模型。它能理解自然語言，並回傳具型別的答案與機率，而不是產生文字或推理說明。工作流程由程式碼掌控；在一般程式碼需要語意理解的地方，由模型提供可程式化的常識。

## 閱讀線上文件

**TypeSafe 線上文件是唯一可信的依據（source of truth），請把閱讀文件當成任務的一部分。**
本技能只提供方向；最新的概念、提示撰寫指引、API 合約、SDK 用法、模型、限制與實作範例都在文件中。

- 從[文件索引](https://docs.typesafe.ai/llms.txt)開始，找出相關頁面與操作手冊。請針對需要的內容閱讀，而不是一次載入整個網站。
- Mintlify 會在頁面路徑後加上 `.md` 時提供 Markdown 版本，例如[如何使用 TypeSafe 建置](https://docs.typesafe.ai/concepts/how-to-build-with-system-one.md)。請順著索引中的連結閱讀；必要時可將沒有副檔名的文件頁面連結改為 `.md`。相對連結請以 `https://docs.typesafe.ai` 為基準解析。
- 撰寫整合程式之前，先閱讀目前的 API 或所選 SDK 的頁面，以及與設計相關的提問指引。若是新的工作流程，也請參考最接近的操作手冊：它往往會展示比通用分類器更好的問題拆解方式。
- 若索引無法取得，請使用下方的直接連結或網站導覽。若抓取 Markdown 失敗，可改抓一般頁面。若完全無法連線存取，請使用可取得的本機文件或已安裝 SDK 的型別定義，說明此限制，並避免憑空捏造與版本相關的細節。

| 任務 | 從這裡開始，再依需要深入細節 |
| --- | --- |
| 理解程式設計模型 | [System One](https://docs.typesafe.ai/concepts/system-one.md)、[建置指南](https://docs.typesafe.ai/concepts/how-to-build-with-system-one.md) |
| 探索可以打造什麼 | [使用案例地圖](https://docs.typesafe.ai/concepts/use-case-map.md)，再從索引找相關操作手冊 |
| 準備輸入與提問 | [狀態（State）](https://docs.typesafe.ai/concepts/state.md)、[原語](https://docs.typesafe.ai/primitives.md)，再閱讀所選原語的頁面 |
| 決定如何處理不確定性 | [信心度（Confidence）](https://docs.typesafe.ai/confidence.md) |
| 撰寫 API 程式碼 | [HTTP API](https://docs.typesafe.ai/api.md)、[Python SDK](https://docs.typesafe.ai/sdk/python.md) 或 [JavaScript SDK](https://docs.typesafe.ai/sdk/javascript.md) |
| 更新舊版整合 | [遷移指南](https://docs.typesafe.ai/migrating-to-v1.md)，以及已安裝 SDK 的最新參考文件 |

## 找出合適的形態

從使用者想要的行為出發：應用程式要顯示、選取、變更或轉交什麼？再從結果往回推，找出它需要哪些判斷。已知的規則、計算、精確查詢與執行動作都留在程式碼中。保留使用者選定的技術堆疊與範圍；只在語意理解有幫助的地方加入 TypeSafe。

在發想點子或選擇架構時，不要只想到分類。以下模式只是起點：圍繞使用者的目標組合各種原語，包括不符合既有做法的構想。

- **路由並填入已知參數。** 一個請求可以同時選出處理函式及其具型別的參數。預先提出各分支所需的問題，只取用相關的答案。可參考[函式呼叫](https://docs.typesafe.ai/cookbooks/function_calling.md)與[推測式扇出（speculative fan-out）](https://docs.typesafe.ai/patterns/fan-out.md)。
- **用選擇取代生成。** 先用程式碼找出候選值或來源片段，再用判斷選出預期的那一個，然後複製或正規化它。程式碼也可以把來源文字組裝成格式化文件或閱讀導引。可參考[數值擷取](https://docs.typesafe.ai/cookbooks/pre_parsed_value_extraction_cookbook.md)與[結構還原](https://docs.typesafe.ai/cookbooks/autoformat.md)。
- **尋找並評判證據。** 檢索候選項目，比較它們與查詢的相關性，並挑選有用的上下文。可參考[重新排序（reranking）](https://docs.typesafe.ai/cookbooks/rerank_typesafe.md)與[階層式分類](https://docs.typesafe.ai/cookbooks/hierarchical_classification.md)。
- **將判斷轉為可重複使用的資料。** 各個維度只評分一次，之後由程式碼或使用者控制項調整權重、門檻、排名與檢視方式。若有已標註的結果，這些訊號也能成為傳統機器學習的特徵。可參考[複合評分](https://docs.typesafe.ai/patterns/composite-scoring.md)與[特徵探索](https://docs.typesafe.ai/cookbooks/autoresearch_feature_discovery.md)。
- **驗證並升級處理。** 針對特定主張或欄位，對照其證據進行檢查；將不確定或未通過的案例交給人工或推理模型處理。可參考[引用檢查](https://docs.typesafe.ai/cookbooks/citation_check.md)與[擷取串接（cascade）](https://docs.typesafe.ai/cookbooks/sde_cascade.md)。
- **回應不斷變化的狀態。** 程式碼可以保存目標與觀察結果，同時由最新的判斷引導下一個有界的步驟。將推論出的狀態與觀察到的事實區分開來，並在將結果套用到已改變的情境之前，先確認它是否仍是最新的。

面對開放式的需求，提出少數幾個最能達成使用者目標的方向，並建議一個起點。面對具體的需求，直接選擇相關模式並開始建置；發想並不是必經的繞路。

## 設計判斷

依答案代表的意義來選擇，再閱讀對應原語的頁面：

| 需求 | 原語 | 重要區別 |
| --- | --- | --- |
| 從既定集合中選一個 | [Choice](https://docs.typesafe.ai/primitives/choice.md) | 選出一個選項；其機率分布用來比較互相競爭的選項 |
| 某個條件是否成立 | [Noul](https://docs.typesafe.ai/primitives/noul.md) | 回傳「是」的機率；沒有另外的信心度；若可能同時適用多個標籤，每個標籤各用一個 Noul |
| 在某個描述維度上的程度 | [Score](https://docs.typesafe.ai/primitives/score.md) | 在有序等級上以機率加權的位置；若要分級排序，對每個項目使用可互相比較的 Score |

為每個問題提供足以作答的相關**狀態（state）**：來源文字、身分、關係、政策與目前事實。當上下文包含多個部分時，優先使用具名的 JSON 欄位。把判斷本身寫在 **instructions（指示）** 中，並在 **criteria（準則）** 中定義可能的答案。問題 ID 是給程式碼用的，不會傳送給模型；請在問題中寫出完整的意義。引用巢狀狀態時，使用反引號包住的路徑，例如 `ticket.messages[0].text`。

每個問題只問一個範圍明確、內容一致的判斷。把彼此獨立且各自有用的維度拆開，但不要拆散正在被判斷的關係。在有限範圍內選擇動作，或依上下文進行詮釋，都是合理的問題；「原子化」不代表只能擷取字面事實，也不代表只能有一句話。簡單的問題用字串即可。當定義、對比、排除條件或範例能讓指示或準則更清楚時，請使用結構化物件或陣列。Score 的各個等級必須描述具體情境，且每個等級都要能獨立理解。

確保所需的答案都在選項之中。當可能沒有任何選項適用時，要加入「無相符」的結果；若「是否存在」本身就是獨立有用的資訊，則另外設一個存在性判斷。在從來源值中挑選時，要檢查候選項目的涵蓋範圍：模型無法選出被遺漏的值。

## 組合與驗證

**針對同一份狀態的獨立問題，請一起提出**，包括有用的推測式問題。它們會平行執行，彼此看不到對方的答案。請明確寫出每個推測式問題的前提；由程式碼取用適用的答案。當需要先取得前一個答案，才能去抓取證據、建構新的狀態或決定下一步的選項時，才需要第二次請求。額外的問題仍然會消耗 token；請實際量測請求預算、成本與端對端延遲。

使用機率與信心度來引導行為，門檻值要依使用者的資料與後果來評估。Choice／Score 的信心度概括的是機率分布的集中程度，而不是整個工作流程的正確性，也不代表可以放行執行動作。Noul 接近 0.5 表示「是」與「否」的機率相近，而不是「中等強度」。多個可接受的替代選項也可能分散機率；對於無傷大雅的偏好選擇，低信心度不一定代表結果無效。未使用分支上的不確定性可以忽略。

讓政策保持明確，讓原始判斷保持可重複使用。加權分數適合可以互相抵補的偏好；「只要有任何嚴重違規」這類規則則需要各自獨立的條件。當證據與問題的意義沒有改變時，調整權重或顯示篩選條件不需要重新推論。具型別的輸出保證的是介面，而不是正確性。System One 模型經過訓練，能做出經過校準的決策；但仍需在目標領域中驗證其表現。

測試具代表性的案例，以及由此產生的應用程式行為。遇到失敗時，檢查確切的狀態、問題、候選項目、答案、組合方式與觀察到的結果。區分證據缺漏、模型錯誤、程式錯誤與服務故障。把操作手冊中的門檻值與示範結果視為有待評估的範例，而不是普遍適用的規則或永久的模型限制。在 Web 應用程式中，API 憑證要保留在伺服器端。
