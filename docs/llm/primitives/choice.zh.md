> 本文件是 [`choice.md`](choice.md)（來源：<https://docs.typesafe.ai/primitives/choice.md>，下載於 2026-09-23）的繁體中文翻譯，供人類閱讀參考。
> 若內容有出入，以線上英文版本為準。程式碼保持原文，範例中的英文字串另附中文對照。
> 原文開頭有一段網站用來產生 Playground 連結的 JavaScript 元件（`TypesafeExample`），與內容無關，本譯文已省略，範例改寫成 JSON 程式碼區塊。

> ## 文件索引
> 完整的文件索引請見：https://docs.typesafe.ai/llms.txt（中文版：[llms.zh.md](../../llms.zh.md)）
> 在深入探索之前，可先用這份索引了解所有可用的頁面。

# Choice

> Choice 是一種 System One 問題類型，用於從既定集合中選出一個選項。答案包含選中的選項、每個選項的機率，以及信心度。

當答案是一組固定選項之一時，就使用 Choice。例如：哪個團隊處理某張工單、某個商品屬於哪個類別，或某段程式碼是用哪種語言寫的。如果答案是光譜上的一個位置，請使用 [Score](score.zh.md)；如果是「是或否」，請使用 [Noul](noul.zh.md)。[選擇問題類型](../primitives.zh.md#選擇問題類型)一節比較了這三者。

Choice 的答案是放在 `choice` 中的選中選項。模型也會在 `probabilities` 中回傳每個選項的機率，並為選中的選項回傳一個 `confidence`（信心度）值。

問題範例：

```
"What programming language is this code written in"
  → options: python, javascript, typescript, go, rust, other

"What type of meeting is this based on the title and description"
  → options: standup, planning, retrospective, one on one, brainstorm, none of the above

"Which product category does this item belong to"
  → options: electronics, clothing, home garden, food and beverage
```

> 中文對照：
> - 「這段程式碼是用哪種程式語言寫的」→ 選項：python、javascript、typescript、go、rust、other（其他）
> - 「根據標題與描述，這是哪一種會議」→ 選項：standup（站立會議）、planning（規劃會議）、retrospective（回顧會議）、one on one（一對一面談）、brainstorm（腦力激盪）、none of the above（以上皆非）
> - 「這個商品屬於哪個類別」→ 選項：electronics（電子產品）、clothing（服飾）、home garden（居家園藝）、food and beverage（食品飲料）

## 請求結構

送到 [TypeSafe API](https://docs.typesafe.ai/api.md) 的 POST 請求內容有特定的結構。頂層有三個欄位：`state`（要評估的內容）、`model`，以及 `questions`（從你自訂的問題 ID 對應到問題物件的 map）。每個 Choice 問題有以下欄位：

* `type`：固定為 `"choice"`。
* `instructions`：模型要回答的問題。
* `criteria`：答案選項，以 map 表示。每個鍵是選項名稱，每個值是該選項的描述。

以下是一個請求，狀態是一間線上鞋店收到的客服工單，問題是應該由哪個團隊處理：

**request**

```json
{
  "state": "My running shoes arrived in the wrong size. Can I swap them for a size 10?",
  "model": "jev-latest",
  "questions": {
    "department": {
      "type": "choice",
      "instructions": "Which team should handle this?",
      "criteria": {
        "returns": "Exchanges, wrong or damaged items",
        "shipping": "Delivery status, delays, lost packages",
        "billing": "Charges, invoices, payment problems"
      }
    }
  }
}
```

> 中文對照：
> - 狀態：「我的跑鞋寄來的尺寸不對。可以換成 10 號嗎？」
> - `department`（部門）：「應該由哪個團隊處理？」
>   - `returns`（退換貨）：換貨、寄錯或損壞的商品
>   - `shipping`（物流）：配送狀態、延誤、包裹遺失
>   - `billing`（帳務）：扣款、發票、付款問題
>
> 原網頁在此範例下方有「Try it in the Playground →」連結，可在 [Playground](https://console.typesafe.ai/playground) 中直接試用。

問題 ID 由你自己決定，這裡是 `department`。答案會在同一個 ID 底下回傳。模型永遠看不到問題 ID。**選項名稱和它們的描述都會傳送給模型**，所以請寫出能把各選項彼此區分開來的描述。

我們的[用戶端 SDK](https://docs.typesafe.ai/sdk.md) 提供具型別的問題。在 Python 中，同一個問題寫成 `Choice`：

```python
from typesafe_sdk import Choice, TypeSafeClient

with TypeSafeClient() as client:
    response = client.system_one(
        state="My running shoes arrived in the wrong size. Can I swap them for a size 10?",
        questions={
            "department": Choice(
                instructions="Which team should handle this?",
                criteria={
                    "returns": "Exchanges, wrong or damaged items",
                    "shipping": "Delivery status, delays, lost packages",
                    "billing": "Charges, invoices, payment problems",
                },
            ),
        },
    )

    print(response.answers["department"].choice)
```

使用 `system_one` 方法或 `https://api.typesafe.ai/v1/systemone` 端點來呼叫 System One 模型。`model` 欄位決定由哪個模型處理請求。[如何使用 TypeSafe 建置](../concepts/how-to-build-with-system-one.zh.md)說明了應該在程式碼的哪個位置呼叫它。

你可以使用我們的[用戶端 SDK](https://docs.typesafe.ai/sdk.md)，或直接呼叫 [HTTP API](https://docs.typesafe.ai/api.md)。如果是由程式開發 Agent 替你撰寫整合程式，請先安裝 [TypeSafe Agent 技能](https://docs.typesafe.ai/agent-skill.md#installation)，讓它知道請求與回應的格式。

> **注意：** `instructions` 與 `criteria` 中的每個項目都可以是字串、物件或陣列。先從字串開始。當描述需要多種指引時，例如某個選項涵蓋什麼、不涵蓋什麼，以及一些範例，就使用物件。請見下方的[結構化的指示與準則](#結構化的指示與準則)，以及 [API 參考](https://docs.typesafe.ai/api.md#param-instructions-1)。

## 回應結構

回應中的 `answers` 對每個問題各有一個項目，放在請求時使用的 ID 底下。以下是上述範例請求的回應：

```json
{
  "model": "jev-1.13.0",
  "answers": {
    "department": {
      "type": "choice",
      "choice": "returns",
      "confidence": 1.0,
      "probabilities": {
        "shipping": 0.0,
        "returns": 1.0,
        "billing": 0.0
      }
    }
  },
  "usage": {
    "input_tokens": 328,
    "output_tokens": 34
  }
}
```

除了 `type` 之外，每個 Choice 答案有三個值：

* `choice`：機率最高的選項。
* `probabilities`：所有選項上的完整機率分布。所有值加總為 1。
* [`confidence`](../confidence.zh.md)：一個 0 到 1 的數字，依據 `probabilities` 的分布方式計算而來。形狀平坦、機率分散在多個選項上，代表低信心度；機率集中在單一選項上形成尖峰，代表高信心度。

這張工單很單純，所以所有機率都在 `returns`（退換貨）上，信心度為 1.0。如果一張工單同時提到尺寸不對和退款沒收到，機率就會分散在 `returns`（退換貨）與 `billing`（帳務）之間，信心度也會下降。

## 良好做法：每次呼叫提出不只一個問題

把你的程式碼可能需要的每個 Choice 問題都放在同一次請求中，而不是每個問題各發一次請求。問題會平行評估。增加問題幾乎不會影響回應時間，程式碼也可以忽略用不到的答案。額外的問題仍然會消耗 token。[一起提出多個問題](../primitives.zh.md#一起提出多個問題)一節有完整說明；下一節會展示在一次呼叫中提出五個 Choice 問題。

同樣的邏輯也適用於單一 Choice 問題內的選項。一個 Choice 問題最多接受 **255 個選項**，每多一個選項只多花幾個 token，所以請給模型完整的團隊、類別或產品清單，而不是精簡過的候選名單。當清單可能無法涵蓋所有輸入時，加上 `other`（其他）或 `none of the above`（以上皆非）選項，讓模型能表達「其他選項都不符合」。

若要透過深層階層或大型分類體系來分類文件，可以逐層串接 Choice 問題。[階層式分類操作手冊](https://docs.typesafe.ai/cookbooks/hierarchical_classification.md)展示了如何對 Choice 機率執行束搜尋（beam search）：在每一層保留最好的 `K` 條候選路徑，而不是貪婪地只走單一路徑。

## 較複雜的範例

上面的基本範例把工單路由到某個團隊。較大的客服系統可能還需要知道退貨原因、配送問題、客戶想要什麼，以及客戶的語氣。

下面的請求針對一張比第一個範例更模稜兩可的工單，提出五個 Choice 問題：這張工單牽涉到三個團隊，而且沒有說明客戶想要什麼。

**request**

```json
{
  "state": "Shoes arrived two weeks late and in the wrong size. Also I see two charges of $120 on my card. What are you going to do about this?",
  "model": "jev-latest",
  "questions": {
    "department": {
      "type": "choice",
      "instructions": "Which team should handle this?",
      "criteria": {
        "returns": "Exchanges, wrong or damaged items",
        "shipping": "Delivery status, delays, lost packages",
        "billing": "Charges, invoices, payment problems"
      }
    },
    "return_reason": {
      "type": "choice",
      "instructions": "If the customer wants to return something, why?",
      "criteria": {
        "wrong_size": "The item doesn't fit",
        "wrong_item": "A different product was delivered",
        "damaged": "The item arrived broken or faulty",
        "changed_mind": "The item is fine, the customer no longer wants it",
        "other": "A return reason that fits none of the above"
      }
    },
    "shipping_issue": {
      "type": "choice",
      "instructions": "If this is a shipping problem, which kind is it?",
      "criteria": {
        "not_delivered": "The package never arrived",
        "delayed": "The package is late but still on its way",
        "wrong_address": "The package went to the wrong place",
        "damaged_in_transit": "The package arrived damaged",
        "other": "A shipping problem that fits none of the above"
      }
    },
    "requested_resolution": {
      "type": "choice",
      "instructions": "What does the customer want to happen?",
      "criteria": {
        "exchange": "Swap the item for a different one",
        "refund": "Money back",
        "replacement": "The same item sent again",
        "information": "Just an answer, no action needed"
      }
    },
    "tone": {
      "type": "choice",
      "instructions": "What is the customer's tone?",
      "criteria": {
        "calm": null,
        "frustrated": null,
        "angry": null
      }
    }
  }
}
```

> 中文對照：
> - 狀態：「鞋子晚了兩週才到，而且尺寸不對。另外我看到信用卡上有兩筆 120 美元的扣款。你們打算怎麼處理？」
> - `department`（部門）：同上一個範例。
> - `return_reason`（退貨原因）：「如果客戶想退貨，原因是什麼？」
>   - `wrong_size`（尺寸不對）：商品不合身
>   - `wrong_item`（寄錯商品）：送來的是不同的產品
>   - `damaged`（損壞）：商品送達時已損壞或有瑕疵
>   - `changed_mind`（改變心意）：商品沒問題，是客戶不想要了
>   - `other`（其他）：不符合以上任何一項的退貨原因
> - `shipping_issue`（配送問題）：「如果這是配送問題，是哪一種？」
>   - `not_delivered`（未送達）：包裹從未送達
>   - `delayed`（延誤）：包裹遲到，但仍在運送途中
>   - `wrong_address`（地址錯誤）：包裹送錯地方
>   - `damaged_in_transit`（運送中損壞）：包裹送達時已損壞
>   - `other`（其他）：不符合以上任何一項的配送問題
> - `requested_resolution`（期望的處理方式）：「客戶希望怎麼處理？」
>   - `exchange`（換貨）：把商品換成另一個
>   - `refund`（退款）：退錢
>   - `replacement`（補寄）：重新寄送同一件商品
>   - `information`（詢問資訊）：只要一個答覆，不需要採取行動
> - `tone`（語氣）：「客戶的語氣如何？」
>   - `calm`（冷靜）、`frustrated`（沮喪）、`angry`（憤怒），描述皆為 `null`
>
> 原網頁在此範例下方有「Try it in the Playground →」連結，可在 [Playground](https://console.typesafe.ai/playground) 中直接試用。

其中兩個 Choice 問題是推測性的：`return_reason` 只有在 `department` 為 `returns` 時才重要，`shipping_issue` 只有在它為 `shipping` 時才重要。`tone` 問題使用 `null` 描述，因為選項名稱本身就已經很清楚了。

TypeSafe 的回應：

```json
{
  "model": "jev-1.13.0",
  "answers": {
    "department": {
      "type": "choice",
      "choice": "returns",
      "confidence": 0.42,
      "probabilities": {
        "shipping": 0.04,
        "billing": 0.35,
        "returns": 0.61
      }
    },
    "return_reason": {
      "type": "choice",
      "choice": "wrong_size",
      "confidence": 1.0,
      "probabilities": {
        "other": 0.0,
        "wrong_size": 1.0,
        "changed_mind": 0.0,
        "damaged": 0.0,
        "wrong_item": 0.0
      }
    },
    "shipping_issue": {
      "type": "choice",
      "choice": "delayed",
      "confidence": 0.67,
      "probabilities": {
        "wrong_address": 0.0,
        "other": 0.26,
        "not_delivered": 0.0,
        "damaged_in_transit": 0.0,
        "delayed": 0.74
      }
    },
    "requested_resolution": {
      "type": "choice",
      "choice": "refund",
      "confidence": 0.2,
      "probabilities": {
        "replacement": 0.34,
        "refund": 0.4,
        "information": 0.02,
        "exchange": 0.24
      }
    },
    "tone": {
      "type": "choice",
      "choice": "frustrated",
      "confidence": 0.76,
      "probabilities": {
        "frustrated": 0.84,
        "angry": 0.16,
        "calm": 0.0
      }
    }
  },
  "usage": {
    "input_tokens": 589,
    "output_tokens": 212
  }
}
```

每個問題都各自對照工單作答：

* `department`（部門）的答案是 `returns`（退換貨），機率 0.61；但因為重複扣款，`billing`（帳務）也有 0.35。這張工單同時屬於兩個團隊，0.42 這個分歧的信心度反映了這一點。
* `return_reason`（退貨原因）是 `wrong_size`（尺寸不對），信心度 1.0。這在意料之中，因為工單裡寫得很清楚。
* `shipping_issue`（配送問題）的答案分散在 `delayed`（延誤）和 `other`（其他）之間。這是推測性的問題，而 `department` 的結果並不是物流，所以程式碼可以忽略它，如下方的範例程式碼所示。
* `requested_resolution`（期望的處理方式）的答案以 0.40 偏向 `refund`（退款），其餘機率大多由 `replacement`（補寄）和 `exchange`（換貨）分攤，信心度為 0.20。重複扣款暗示要退錢，尺寸不對暗示要換貨，而客戶始終沒說他想要哪一種。
* `tone`（語氣）的答案是 `frustrated`（沮喪），機率 0.84，信心度 0.76。

下面的範例程式碼只讀取它需要的答案、忽略其餘的答案，並把低信心度的答案視為「應該先詢問、而不是直接行動」的理由：

```python
from typesafe_sdk import Choice, TypeSafeClient

TRIAGE_QUESTIONS = {
    "department": Choice(
        instructions="Which team should handle this?",
        criteria={
            "returns": "Exchanges, wrong or damaged items",
            "shipping": "Delivery status, delays, lost packages",
            "billing": "Charges, invoices, payment problems",
        },
    ),
    "return_reason": Choice(
        instructions="If the customer wants to return something, why?",
        criteria={
            "wrong_size": "The item doesn't fit",
            "wrong_item": "A different product was delivered",
            "damaged": "The item arrived broken or faulty",
            "changed_mind": "The item is fine, the customer no longer wants it",
            "other": "A return reason that fits none of the above",
        },
    ),
    "shipping_issue": Choice(
        instructions="If this is a shipping problem, which kind is it?",
        criteria={
            "not_delivered": "The package never arrived",
            "delayed": "The package is late but still on its way",
            "wrong_address": "The package went to the wrong place",
            "damaged_in_transit": "The package arrived damaged",
            "other": "A shipping problem that fits none of the above",
        },
    ),
    "requested_resolution": Choice(
        instructions="What does the customer want to happen?",
        criteria={
            "exchange": "Swap the item for a different one",
            "refund": "Money back",
            "replacement": "The same item sent again",
            "information": "Just an answer, no action needed",
        },
    ),
    "tone": Choice(
        instructions="What is the customer's tone?",
        criteria={"calm": None, "frustrated": None, "angry": None},
    ),
}


def triage(ticket: str) -> None:
    with TypeSafeClient() as client:
        response = client.system_one(
            state=ticket,
            questions=TRIAGE_QUESTIONS,
        )
    answers = response.answers

    department = answers["department"]
    if department.confidence < 0.3:
        # Not clear which team to send to. Let a person decide.
        send_to_manual_triage(ticket)
        return

    if department.choice == "returns":
        # return_reason answer is only used here
        assign(ticket, team="returns", issue=answers["return_reason"].choice)
    elif department.choice == "shipping":
        # shipping_issue answer is only used here
        assign(ticket, team="shipping", issue=answers["shipping_issue"].choice)
    else:
        assign(ticket, team="billing")

    # A second team with a real share of the probability gets a copy
    for team, probability in department.probabilities.items():
        if team != department.choice and probability > 0.25:
            notify(ticket, team=team)

    resolution = answers["requested_resolution"]
    if resolution.confidence < 0.5:
        # The customer hasn't said what they want. Ask, don't guess.
        ask_customer_what_they_want(ticket)
    elif resolution.choice == "refund":
        flag_for_refund_approval(ticket)

    if answers["tone"].choice == "angry":
        flag_for_senior_agent(ticket)
```

> 中文對照（程式邏輯與註解）：
> - `department` 信心度 < 0.3：「不清楚該送給哪個團隊。交給人決定。」→ 送交人工分流（`send_to_manual_triage`），並結束。
> - `returns`（退換貨）：「`return_reason` 的答案只在這裡使用」→ 指派給退換貨團隊，問題類型為退貨原因。
> - `shipping`（物流）：「`shipping_issue` 的答案只在這裡使用」→ 指派給物流團隊，問題類型為配送問題。
> - 其他情況 → 指派給帳務團隊。
> - 「機率佔有實質比例的第二個團隊，會收到一份副本」→ 對於非選中、但機率 > 0.25 的團隊發出通知（`notify`）。
> - `requested_resolution` 信心度 < 0.5：「客戶沒說他想要什麼。要問，不要猜。」→ 詢問客戶想要什麼（`ask_customer_what_they_want`）；若是有把握的 `refund`（退款），則標記送交退款審核（`flag_for_refund_approval`）。
> - 語氣為 `angry`（憤怒）→ 標記交由資深客服處理（`flag_for_senior_agent`）。

以上面的工單來說，這段程式碼會把工單指派給退換貨團隊、問題類型為 `wrong_size`（尺寸不對）；因為帳務團隊 0.35 的比例超過 0.25 的門檻，會寄一份副本給帳務團隊；又因為處理方式的信心度 0.20 低於 0.5，所以會詢問客戶想要什麼。這段程式碼沒有使用 `shipping_issue` 的答案。

一次請求、五個答案，路由邏輯就只是一般的 `if` 敘述。如果之後你需要知道客戶使用的語言，或工單是關於哪個產品，只要在 `TRIAGE_QUESTIONS` 再加一個 Choice 問題即可，請求次數仍然只有一次。

[智慧家庭助理示範](https://docs.typesafe.ai/demos/smart-home.md)在一次呼叫中，用一長串 Choice 問題評估每一個使用者請求：請求類別、房間、裝置，以及動作。對任何一個請求來說，大多數問題都無關，程式碼會直接忽略它們。

## 結構化的指示與準則

先從每個選項一行描述開始。當兩個選項很相似、模型一直把它們搞混時，就把每個選項的描述從字串改成物件。在物件中放入幾個欄位：這個選項涵蓋什麼、哪些內容其實屬於相鄰的選項，以及幾個範例輸入。

下面這兩個答案選項 `return_policy`（退貨政策）和 `return_status`（退貨進度）很容易混淆。關於這兩者的工單都可能提到退貨和退款，所以每個選項都寫明了自己「不適用」於什麼。

**request**

```json
{
  "state": "I sent the shoes back a week ago. When do I get my money?",
  "model": "jev-latest",
  "questions": {
    "return_topic": {
      "type": "choice",
      "instructions": {
        "question": "Which returns topic is the customer asking about?",
        "focus": "Classify the information the customer wants."
      },
      "criteria": {
        "return_policy": {
          "what": "Whether and how an item can be returned",
          "not_for": "Progress of a return already sent",
          "examples": [
            "Can I return shoes I've worn once?",
            "How long do I have to return an order?"
          ]
        },
        "return_status": {
          "what": "Progress of a return already sent",
          "not_for": "Whether and how an item can be returned",
          "examples": [
            "Has my return arrived yet?",
            "When will my refund be paid?"
          ]
        }
      }
    }
  }
}
```

> 中文對照：
> - 狀態：「我一週前就把鞋子寄回去了。什麼時候可以拿到退款？」
> - `instructions`：
>   - `question`：「客戶詢問的是哪一個退貨相關主題？」
>   - `focus`：「依客戶想要的資訊來分類。」
> - `return_policy`（退貨政策）：
>   - `what`：商品能否退貨、如何退貨
>   - `not_for`：已寄出退貨的處理進度
>   - `examples`：「穿過一次的鞋子可以退嗎？」、「訂單有多久的退貨期限？」
> - `return_status`（退貨進度）：
>   - `what`：已寄出退貨的處理進度
>   - `not_for`：商品能否退貨、如何退貨
>   - `examples`：「我的退貨寄到了嗎？」、「退款什麼時候會入帳？」
>
> 原網頁在此範例下方有「Try it in the Playground →」連結，可在 [Playground](https://console.typesafe.ai/playground) 中直接試用。

回應是 `return_status`（退貨進度），信心度 1.0：

```json
{
  "model": "jev-1.13.0",
  "answers": {
    "return_topic": {
      "type": "choice",
      "choice": "return_status",
      "confidence": 1.0,
      "probabilities": {
        "return_policy": 0.0,
        "return_status": 1.0
      }
    }
  },
  "usage": {
    "input_tokens": 407,
    "output_tokens": 32
  }
}
```

`question`、`focus`、`what`、`not_for` 與 `examples` 這些欄位名稱**不是** API 的一部分，也沒有任何一個是保留字。它們由你自訂，就像你自訂選項名稱一樣。模型會同時看到名稱與值，所以請使用簡短、能標示後面內容的名稱。
