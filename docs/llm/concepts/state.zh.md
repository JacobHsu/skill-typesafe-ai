> 本文件是 [`state.md`](state.md)（來源：<https://docs.typesafe.ai/concepts/state.md>，下載於 2026-09-23）的繁體中文翻譯，供人類閱讀參考。
> 若內容有出入，以線上英文版本為準。程式碼保持原文，範例中的英文字串另附中文對照。

> ## 文件索引
> 完整的文件索引請見：https://docs.typesafe.ai/llms.txt（中文版：[llms.zh.md](../../llms.zh.md)）
> 在深入探索之前，可先用這份索引了解所有可用的頁面。

# 狀態（State）

> 什麼是狀態、如何組織它，以及如何提供 System One 模型所需的上下文。

**狀態（state）**是你要求 System One 模型評估的內容。它可以是一則客服訊息、一段文字，或是你應用程式目前的狀態。你把它放在 API 請求的 `state` 欄位中，與你想要得到答案的問題一起送出。

每次請求會針對一份狀態評估一個或多個問題。所有問題看到的是同一份狀態，並且各自獨立評估。你可以在同一次請求中混合使用 [Choice](../primitives/choice.zh.md)、[Score](../primitives/score.zh.md) 與 [Noul](../primitives/noul.zh.md) 問題。

## 狀態可以是簡單的字串，也可以是結構化的 JSON 值

最簡單的狀態就是一個純字串：

```python
state = "My card was charged twice."
```

> 中文對照：「我的卡被扣款了兩次。」

狀態也可以是 JSON 物件或陣列，裡面放入相關的上下文、範例，以及其他能幫助模型回答對應問題的資訊。可以把狀態想成：在請一組專家做出判斷之前，你會先交給他們看的資料。在 Python 中，直接把對應的字串、字典或串列傳給 `client.system_one(state=...)` 即可。

| 格式 | 適用情境 | 範例 |
| --- | --- | --- |
| 字串 | 一則訊息、一篇文章或一段文字 | `"My card was charged twice."` |
| 物件 | 具名欄位、相關紀錄或應用程式狀態 | `{"message": "My card was charged twice.", "order_id": "A-104"}` |
| 陣列 | 一連串的訊息或紀錄 | `["Hi", "My customer number is TS1337.", "My card was charged twice."]` |

> 中文對照（陣列範例）：「嗨」、「我的客戶編號是 TS1337。」、「我的卡被扣款了兩次。」

大多數請求都建議使用物件，這樣狀態的每個部分都有描述性的名稱，彼此之間的關係也能保持清楚。當使用情境很簡單、只需要一段文字時，使用字串即可。

> **注意：** Jev 只接受文字。狀態必須是字串、JSON 物件或文字值組成的陣列。圖片、音訊與影片（目前）尚不支援。Jev 的主要訓練語言是英文；其他語言（包括中日韓文字）也可以輸入，但目前準確度較低，請見[模型](https://docs.typesafe.ai/models.md#language-support)。

**以一段客服對話作為狀態**

```json
{
  "ticket": {
    "subject": "Duplicate charge",
    "messages": [
      {"from": "customer", "text": "I was charged twice for order A-104. Please refund the duplicate."},
      {"from": "support", "text": "We are checking the charges."}
    ]
  },
  "order": {
    "id": "A-104",
    "charges": [
      {"amount_usd": 49, "status": "captured"},
      {"amount_usd": 49, "status": "captured"}
    ]
  },
  "refund_policy": "Duplicate charges are eligible for a refund."
}
```

> 中文對照：
> - `ticket.subject`：「重複扣款」
> - 客戶訊息：「訂單 A-104 被扣款了兩次。請退還重複的那筆。」
> - 客服訊息：「我們正在確認這些扣款。」
> - `order.charges`：兩筆各 49 美元、狀態皆為 `captured`（已請款）的扣款
> - `refund_policy`：「重複扣款符合退款資格。」

這個物件是**一份**狀態，即使它包含了一段對話、一筆訂單和一項政策。當決策需要比對這些部分時，就把相關資訊放在一起。

## 將內容與問題分開

狀態包含內容與佐證事實；[問題](../primitives.zh.md)則定義模型應該針對這些資料做出哪些判斷。例如，把退款請求與政策放在狀態中，再分別詢問「客戶是否要求退款」以及「政策是否支持退款」。

關於指示（instructions）、準則（criteria）、問題類型，以及如何針對同一份狀態提出多個問題，請見[原語（問題）](../primitives.zh.md)。

請求的結構定義請見 [API 參考](https://docs.typesafe.ai/api.md)；安裝方式、具型別的輸入與回應處理，請見[用戶端 SDK](https://docs.typesafe.ai/sdk.md)。
