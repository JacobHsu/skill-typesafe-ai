> 本文件是 [`system-one.md`](system-one.md)（來源：<https://docs.typesafe.ai/concepts/system-one.md>，下載於 2026-09-23）的繁體中文翻譯，供人類閱讀參考。
> 若內容有出入，以線上英文版本為準。

> ## 文件索引
> 完整的文件索引請見：https://docs.typesafe.ai/llms.txt（中文版：[llms.zh.md](../../llms.zh.md)）
> 在深入探索之前，可先用這份索引了解所有可用的頁面。

# System One

> System One 模型為軟體做出快速、結構化的決策。Jev 是 TypeSafe 的旗艦模型，也是第一個 System One 模型。

System One 模型是一類 AI 模型，專為做出快速、結構化、可讓軟體直接使用的決策而打造。System One 模型會評估一份[狀態（state）](state.zh.md)，並回傳具型別的答案與機率。

Jev 是 TypeSafe 的旗艦模型，也是第一個 System One 模型。

和 LLM 一樣，System One 模型能理解自然語言輸入。但它回傳的是具型別的決策與機率，而不是生成的文字。

> **注意：** Jev 目前只接受文字輸入。它可以評估字串、JSON 物件與文字陣列。圖片、音訊與影片（目前）尚不支援。

## 與 LLM 有何不同

System One 模型是為了做出經過校準（calibrated）的決策而訓練的：它們的機率會對照實際結果進行最佳化，以反映不確定性。校準程度是以一群預測整體來衡量的；它並不保證任何單一答案一定正確。

System One 模型不會撰寫回覆、產生程式碼，也不會生成推理過程的說明。可能的答案由你透過[原語（primitives）](../primitives.zh.md)來定義：

| 原語 | 問題 | 答案空間範例 | 輸出範例 |
| --- | --- | --- | --- |
| [Choice](../primitives/choice.zh.md) | 哪個團隊應該處理這張工單？ | `billing`、`technical` 或 `account` | `choice: "billing"` |
| [Score](../primitives/score.zh.md) | 這位客戶有多沮喪？ | 0 = 冷靜，1 = 沮喪，2 = 非常沮喪 | `score: 1.4` |
| [Noul](../primitives/noul.zh.md) | 這則訊息是否要求退款？ | 真或假 | `noul: 0.95` |

以上僅為示意用的設定與數值。可用的設定選項與完整的回應欄位，請見各原語的頁面。

想了解 System One 模型如何運作與如何訓練，請閱讀 [AI 入門](https://docs.typesafe.ai/introduction/machine-learning-primer.md)。

> **注意：** System One 這個名稱來自 Daniel Kahneman 在《快思慢想》（*Thinking, Fast and Slow*）一書中推廣的概念。系統一（System 1）的思考快速而直覺；系統二（System 2）則較慢、較審慎。這裡強調的是快速、聚焦的判斷。

## 在更大工作流程中的快速判斷

以退款請求為例，你的應用程式可以：

1. 建立一份狀態，包含客戶的訊息、相關的交易紀錄，以及退款政策。
2. 一起提出多個彼此獨立的問題：客戶是否要求退款、證據是否顯示有重複扣款，以及政策是否支持退款。
3. 在程式碼中將這些答案與確定性的檢查結合，再把案件路由去執行處理或送交審核。

當你看過原語實際運作後，就可以把它們組合成更大的系統。由於 System One 模型回傳的是具型別、受限制的輸出，而不是自由格式的文字，你的程式碼可以檢查並組合這些答案，形成可預測的工作流程。完整的工作流程請見[如何使用 TypeSafe 建置](how-to-build-with-system-one.zh.md)。

System One 模型的答案也包含[信心度（confidence）](../confidence.zh.md)，讓你能決定何時採取行動、何時升級交給人工或推理模型處理。

## 呼叫 System One 模型

你可以透過我們的[用戶端 SDK](https://docs.typesafe.ai/sdk.md)，或 [HTTP API](https://docs.typesafe.ai/api.md) 中的 `POST /v1/systemone` 來呼叫 System One 模型。`model` 欄位決定由哪個模型處理請求。本文件中的範例使用 `jev-latest`，這也是 SDK 的預設值。可用的模型、價格與別名，請見[模型](https://docs.typesafe.ai/models.md)。

先從[狀態](state.zh.md)開始準備輸入，再到[原語（問題）](../primitives.zh.md)探索你可以提出的問題類型。
