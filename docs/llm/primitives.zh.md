> 本文件是 [`primitives.md`](primitives.md)（來源：<https://docs.typesafe.ai/primitives.md>，下載於 2026-09-23）的繁體中文翻譯，供人類閱讀參考。
> 若內容有出入，以線上英文版本為準。程式碼保持原文，範例中的英文字串另附中文對照。
> 原文開頭有一段網站用來產生 Playground 連結的 JavaScript 元件（`TypesafeExample`），與內容無關，本譯文已省略。

> ## 文件索引
> 完整的文件索引請見：https://docs.typesafe.ai/llms.txt（中文版：[llms.zh.md](../llms.zh.md)）
> 在深入探索之前，可先用這份索引了解所有可用的頁面。

# 原語（問題）

> TypeSafe 的三種問題類型（Choice、Score、Noul）、它們回傳的具型別答案、如何在三者之間選擇，以及如何一次提出多個問題。

TypeSafe 的原語（primitives）是小型、具型別的積木，由你在程式碼中組合。它們成對出現：**問題（question）**定義一個要由 [System One 模型](concepts/system-one.zh.md)針對某份[狀態（state）](concepts/state.zh.md)做出的判斷，而**答案（answer）**則是回傳的具型別值。你在程式碼中組合這些答案來做出決策。問題共有三種類型，每一種回傳不同形態的答案。

| 類型 | 回答什麼 | 回傳 |
| --- | --- | --- |
| [Choice](primitives/choice.zh.md) | 這些選項中的哪一個？ | `choice`、`probabilities`、`confidence` |
| [Score](primitives/score.zh.md) | 哪一個等級？ | `score`、`legend`、`probabilities`、`confidence` |
| [Noul](primitives/noul.zh.md) | 這是真的嗎？ | `noul`（0 到 1） |

你可以只問一個問題，也可以一起送出多個問題。同一請求中的每個問題看到的都是同一份狀態，各自獨立評估，並在你所指定的 ID 底下回傳具型別的答案。

## 每個問題只要求一個直覺判斷

System One 模型是為快速、聚焦的判斷而打造的。請詢問那種「知識豐富的人在拿到正確上下文後，一秒鐘就能做出」的判斷。「這則訊息是否傳達了緊急性？」是個好問題；「分析這則訊息並決定最佳的處理方式」則不是。後者需要慢速推理，這正是一個訊號：應該把任務拆解成小問題，再在程式碼中組合答案。

如果你想要的判斷取決於多個彼此獨立的因素，就針對每個因素分別提問，再用你自己的邏輯組合答案。與其問「替這份新創提案評分」，不如分別詢問市場規模、技術可行性與差異化程度，再依它們的相對重要性在程式碼中加權。當優先順序改變時，只需修改權重的數值，而不必重寫提示詞。[一起提出多個問題](#一起提出多個問題)一節會示範做法。

## 定義一個問題

每個問題都有一個 ID、一個 `type` 和 `instructions`。Choice 與 Score 問題還需要 `criteria`，用來定義 Choice 問題的選項，或 Score 的等級。Noul 問題則可以選擇性地接受 `criteria`，用來補充說明「是」與「否」各代表什麼。

* **ID**：由你自訂的鍵，例如 `refund_requested`。它用來在回應中識別答案。
* **`type`**：`choice`、`score` 或 `noul` 其中之一。
* **`instructions`**：你針對狀態所提出的問題，也就是你的評估邏輯所在之處。請寫成清楚、具體的問句，或寫成一個讓模型判斷的敘述。大多數問題用字串就夠了。它也可以是物件或陣列，把問題放在一個欄位、把它所參照的資料放在其他欄位；請見[在問題中使用結構](concepts/how-to-build-with-system-one.zh.md#在問題中使用結構)。
* **`criteria`**：可能的答案。Choice 問題是選項的對應表（map），Score 是有序的等級清單，Noul 則是選擇性的「是／否」說明。各問題類型的頁面會說明其格式。

這個問題詢問客戶是否要求退款：

```python
from typesafe_sdk import Noul

questions = {
    "refund_requested": Noul(
        instructions="Does the customer request a refund?",
    ),
}
```

> 中文對照：`instructions`＝「客戶是否要求退款？」

> **提示：** 問題 ID 是給你的程式碼用的，**不會**傳送給模型。即使 ID 看起來已經不言自明，也請在 `instructions` 中寫出完整的問題。

## 選擇問題類型

依你需要的答案形態來選擇類型。

* **Choice** 適用於答案是一組已知選項之一、且選項之間沒有順序的情況：把工單路由到某個部門、分類文件類型、偵測程式語言。請列出完整的選項清單；當清單可能無法涵蓋所有輸入時，加上 `other`（其他）或 `none of the above`（以上皆非）選項。

* **Score** 適用於答案落在一個連續光譜上，且你能描述光譜上每個點代表什麼的情況：錯誤的嚴重程度、客戶的沮喪程度、技能水準。等級由你定義，模型會回傳在這些等級上的位置。

* **Noul** 適用於清楚的是非題，且機率本身就是有用訊號的情況：這則訊息是否包含個人身分資訊、客戶是否要求退款、這份履歷是否提到分散式系統。

> **注意：** 是非判斷用 Noul，測量在光譜上的位置用 Score。「這位應徵者的 Python 能力強嗎？」需要對「強」有清楚的定義。Noul 值為 0.5 代表模型認為「是」與「否」的機率相等，**不代表**應徵者的技能是中等水準。定義不清楚時，這個機率就很難解讀。
>
> 如果你想測量技能水準，請使用定義好等級的 Score，例如：沒有經驗、略有接觸、每天使用、深厚專業。如果你需要的是是非決策，請清楚定義條件，例如「履歷中是否說明應徵者曾在工作中使用 Python？」

如果兩種類型似乎都適用，優先選擇程式碼能直接依據其答案行動的那一種。在 `refund`（退款）、`rebook`（改訂）、`information`（詢問資訊）之間做 Choice，可以直接對應到三條程式路徑；客戶沮喪程度的 Score 可以對應到一個門檻值；Noul 則對應到一個 `if`。

## 回傳的內容

答案本身也是原語。每種問題類型都會回傳一個具型別的值，你的程式碼可以拿它來比較、套用門檻、排序、傳入後續邏輯，或放進後續請求的狀態中（請見[當一個問題依賴另一個問題時](#當一個問題依賴另一個問題時)）。

| 類型 | 答案欄位 | 如何解讀 |
| --- | --- | --- |
| Choice | `choice`、`probabilities`、`confidence` | `choice` 是選中的選項。`probabilities` 是所有選項上的機率分布。`confidence` 概括這個分布有多集中。 |
| Score | `score`、`legend`、`probabilities`、`confidence` | `score` 是在你所定義等級上的位置，可以落在兩個等級之間。`legend` 以編號重列各等級。`probabilities` 是各等級上的機率分布。 |
| Noul | `noul` | 答案為「是」的機率。接近 1 是強烈的「是」，接近 0 是強烈的「否」，接近 0.5 則是不確定。Noul 沒有另外的 `confidence`。 |

這些答案有兩個特性，使它們可以被組合：

* **每個答案都被限制在你提供的選項之內。** 模型回傳的是你的選項或等級上的機率分布，絕不會是範圍外的值。你的程式碼永遠不需要從生成的文字中撈出數值。
* **每個答案都是獨立的。** 一個問題的答案不會成為另一個問題的隱藏上下文。你可以新增或移除問題，而不會改變其他問題的結果。

[信心度](confidence.zh.md)一頁說明 `confidence` 如何從 `probabilities` 推導而來，以及如何用它決定何時自動處理、何時升級交給人工。

## 參照特定欄位

被評估的內容，也就是[狀態](concepts/state.zh.md)，通常是包含多個部分的 JSON 物件：一段對話、一筆紀錄、一項政策。當某個問題只針對其中一部分時，請在 `instructions` 中用「點號加索引」的路徑指出它的鍵，**並包含反引號**。這樣模型就知道要判斷狀態的哪個部分。

以「狀態」頁面中的客服對話為例：

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

> 中文對照請見[狀態](concepts/state.zh.md)頁面的同一個範例。

以下兩個問題透過路徑，指向客戶的訊息、政策與扣款紀錄：

```python
questions = {
    "refund_requested": {
        "type": "noul",
        "instructions": "Does `ticket.messages[0].text` request a refund?",
    },
    "policy_supports_refund": {
        "type": "noul",
        "instructions": (
            "Does `refund_policy` support the refund requested "
            "in `ticket.messages[0].text`, given `order.charges`?"
        ),
    },
}
```

> 中文對照：
> - `refund_requested`：「`ticket.messages[0].text` 是否要求退款？」
> - `policy_supports_refund`：「根據 `order.charges`，`refund_policy` 是否支持 `ticket.messages[0].text` 中所要求的退款？」

明確的路徑能清楚表明結構化狀態中的哪些部分應該影響每個判斷。如何組織輸入，請見[狀態](concepts/state.zh.md)。

## 一起提出多個問題

針對同一份狀態的所有問題，請放在同一次請求中送出。你可以自由混合各種問題類型。System One 模型會平行評估請求中的每個問題。增加問題幾乎不會影響回應時間，只會多花額外問題本身的 token，而這很便宜。問一個你可能用不到的問題，成本幾乎是零。

這個請求一次完成：分類客戶訊息、檢查緊急性，並替沮喪程度評分：

**request**

```json
{
  "state": "Our API integration started returning 500 errors on every request about 20 minutes ago, and we can't process any customer orders until this is fixed.",
  "questions": {
    "department": {
      "type": "choice",
      "instructions": "Which team should handle this",
      "criteria": {
        "billing": "Payment or subscription issues",
        "technical": "Bugs or integration problems",
        "sales": "Pricing or account questions"
      }
    },
    "is_urgent": {
      "type": "noul",
      "instructions": "The message conveys urgency or time-sensitivity"
    },
    "frustration": {
      "type": "score",
      "instructions": "How frustrated the customer appears",
      "criteria": [
        "Calm, just stating facts",
        "Frustrated but civil",
        "Very angry, strong language"
      ]
    }
  }
}
```

> 中文對照：
> - 狀態：「我們的 API 整合大約 20 分鐘前開始，每個請求都回傳 500 錯誤，在修好之前我們無法處理任何客戶訂單。」
> - 三個問題與[快速開始](introduction/quickstart.zh.md)中的範例相同：`department`（應由哪個團隊處理）、`is_urgent`（是否傳達緊急性）、`frustration`（客戶的沮喪程度）。
>
> 原網頁在此範例下方有「Try it in the Playground →」連結，可在 [Playground](https://console.typesafe.ai/playground) 中直接試用。

我們的[用戶端 SDK](https://docs.typesafe.ai/sdk.md) 提供具型別的問題與答案。在 Python 中，把由 `Choice`、`Noul`、`Score` 物件組成的 `questions` 字典傳給 `client.system_one(...)` 即可。這個請求只送出一次工單與退款政策，就為每個問題取得一個具型別的答案：

```python
from typesafe_sdk import Choice, Noul, Score, TypeSafeClient

state = {
    "ticket_message": "My flight was cancelled. Can I get a refund?",
    "refund_policy": "Cancelled flights are eligible for a full refund.",
}

with TypeSafeClient() as client:
    response = client.system_one(
        state=state,
        questions={
            "refund_requested": Noul(
                instructions="Does `ticket_message` request a refund?",
            ),
            "request_type": Choice(
                instructions="What is the main request in `ticket_message`?",
                criteria={
                    "refund": "The customer wants money returned.",
                    "rebooking": "The customer wants a replacement flight.",
                    "information": "The customer is asking for information only.",
                },
            ),
            "frustration": Score(
                instructions="How frustrated does the customer appear in `ticket_message`?",
                criteria=[
                    "Calm and neutral.",
                    "Concerned but civil.",
                    "Very angry or using strong language.",
                ],
            ),
        },
    )

print(response.answers["refund_requested"].noul)
print(response.answers["request_type"].choice)
print(response.answers["frustration"].score)
```

> 中文對照：
> - 狀態：`ticket_message`＝「我的航班被取消了。我可以退款嗎？」；`refund_policy`＝「被取消的航班符合全額退款資格。」
> - `refund_requested`（Noul）：「`ticket_message` 是否要求退款？」
> - `request_type`（Choice）：「`ticket_message` 中的主要請求是什麼？」
>   - `refund`：客戶想要拿回款項。
>   - `rebooking`：客戶想要替代航班。
>   - `information`：客戶只是在詢問資訊。
> - `frustration`（Score）：「客戶在 `ticket_message` 中看起來有多沮喪？」
>   - 等級 0：冷靜、中性。
>   - 等級 1：有些擔心但仍有禮貌。
>   - 等級 2：非常憤怒或用詞強烈。

各語言的安裝與用法，請見[用戶端 SDK](https://docs.typesafe.ai/sdk.md)。

### 提出推測性的問題

把你的程式碼可能需要的每個問題都問出來，包括那些只對部分輸入才重要的問題，再由程式碼決定要使用哪些答案。如果某張工單最後其實不是錯誤回報，就忽略嚴重程度的答案。我們稱之為[推測式扇出（Speculative fan-out）](patterns/fan-out.zh.md)模式。[平行提問操作手冊](https://docs.typesafe.ai/cookbooks/parallel_questions.md)顯示，把 13 個問題批次放進一次呼叫，比分成 13 次呼叫便宜 11.5 倍、快 9.6 倍，且答案沒有任何改變。

> **提示：** 程式開發 Agent 比人更容易陷入「每次呼叫只問一個問題」的習慣。[TypeSafe Agent 技能](https://docs.typesafe.ai/agent-skill.md#installation)會告訴你的 Agent 在每次呼叫中放入多個問題，包括只對部分輸入才重要的問題。

### 把複雜的判斷拆成多個問題

取決於多項因素的判斷，最好拆成「每項因素一個問題」。在程式碼中組合答案，並依相對重要性給每個答案一個權重。權重由你決定。當組合後的結果與你的團隊會做出的決定不一致時，就在程式碼中調整權重再執行一次。因為這些問題在同一次請求中平行執行，增加問題幾乎不會影響回應時間；拆分只會多花一些問題本身的 token。

例如，工單優先順序可以由三個 Score 問題構成：錯誤有多嚴重、客戶有多沮喪，以及這份回報提供給工程師多少可用資訊。Score 頁面中的[將複雜判斷拆成多個 Score](primitives/score.zh.md#將複雜判斷拆成多個-score-問題) 一節，會逐步說明這個請求，以及將答案正規化並加權的程式碼。這個技巧稱為[複合評分（Composite scoring）](patterns/composite-scoring.zh.md)模式。

### 當一個問題依賴另一個問題時

同一請求中的問題彼此獨立：一個答案不會成為另一個問題的上下文。如果後面的判斷依賴前面的答案，就在程式碼中發出第二次請求。只有當你的程式碼在拿到第一個答案之前**無法**建構第二次請求時，這種依賴才是真的：例如需要用答案去抓取更多資料放進狀態、決定狀態由哪些內容組成，或挑選下一個問題的選項。否則，請把問題一起提出，再在程式碼中組合答案。

兩次請求是例外，而不是常態。如果第二次請求的問題其實可以針對原本的狀態提出，那就在第一次請求中一起問，再讓程式碼忽略用不到的答案。有三個操作手冊是出於真正的理由才發出第二次請求：

- [技能建議](https://docs.typesafe.ai/cookbooks/skill_suggestion.md)：先在一次請求中排序 182 個技能，再抓取前三名的完整內容，根據這些更好的證據重新判斷。
- [結構還原](https://docs.typesafe.ai/cookbooks/autoformat.md)：先詢問每個換行是否切斷了句子，依據這些答案把行合併成區塊，再替這些區塊分類；而這些區塊在第一次請求回答之前根本還不存在。
- [階層式分類](https://docs.typesafe.ai/cookbooks/hierarchical_classification.md)：用每一次 Choice 的答案，決定下一次請求要提供哪些選項。

關於如何把工作流程拆解成聚焦的判斷，請見[如何使用 TypeSafe 建置](concepts/how-to-build-with-system-one.zh.md)。

## 下一步

- **[Choice](primitives/choice.zh.md)**：從固定清單中選出一個選項。
- **[Score](primitives/score.zh.md)**：依有序的等級替狀態評分。
- **[Noul](primitives/noul.zh.md)**：取得某個敘述為真的機率。

想了解這些原語如何組合成系統架構，請前往[模式](https://docs.typesafe.ai/patterns.md)。
