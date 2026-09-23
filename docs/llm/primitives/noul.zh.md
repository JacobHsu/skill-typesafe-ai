> 本文件是 [`noul.md`](noul.md)（來源：<https://docs.typesafe.ai/primitives/noul.md>，下載於 2026-09-23）的繁體中文翻譯，供人類閱讀參考。
> 若內容有出入，以線上英文版本為準。程式碼保持原文，範例中的英文字串另附中文對照。
> 原文開頭有一段網站用來產生 Playground 連結的 JavaScript 元件（`TypesafeExample`），與內容無關，本譯文已省略，範例改寫成 JSON 程式碼區塊。

> ## 文件索引
> 完整的文件索引請見：https://docs.typesafe.ai/llms.txt（中文版：[llms.zh.md](../../llms.zh.md)）
> 在深入探索之前，可先用這份索引了解所有可用的頁面。

# Noul

> Noul 問題要求 TypeSafe 模型評估一個是非題，並回傳答案為「是」的機率。

當答案是「是或否」時，就使用 Noul。例如：這則訊息是否要求退款、這份履歷是否提到分散式系統、這則留言是否包含個人資料。如果答案是多個選項之一，請使用 [Choice](choice.zh.md)；如果是光譜上的一個位置，請使用 [Score](score.zh.md)。[選擇問題類型](../primitives.zh.md#選擇問題類型)一節比較了這三者。

Noul 的答案是一個數字，代表答案為「是」的機率：0 代表否，1 代表是。

## 請求結構

送到 [TypeSafe API](https://docs.typesafe.ai/api.md) 的 POST 請求內容，和其他問題類型一樣有三個頂層欄位：`state`（要評估的內容）、`model`，以及 `questions`。每個 Noul 問題有以下欄位：

* `type`：固定為 `"noul"`。
* `instructions`：模型要回答的是非題，或是一個讓模型判斷真假的敘述。
* `criteria`：選填。一個物件，以 `true` 與 `false` 分別描述「是」與「否」代表什麼。

以下是一個請求，狀態是一則客服訊息，兩個問題分別是：客戶是否想找真人，以及客戶之前是否聯絡過客服：

**request**

```json
{
  "state": "I have asked three times now. Can I please just talk to a real person?",
  "model": "jev-latest",
  "questions": {
    "is_human_escalation": {
      "type": "noul",
      "instructions": "Is the customer asking for a human agent?"
    },
    "is_repeat_contact": {
      "type": "noul",
      "instructions": "Has the customer contacted support about this before?",
      "criteria": {
        "true": "Mentions a prior attempt, ticket, or that they have asked before",
        "false": "No sign of any previous contact"
      }
    }
  }
}
```

> 中文對照：
> - 狀態：「我已經問了三次了。可以拜託讓我直接跟真人說話嗎？」
> - `is_human_escalation`（是否要求轉真人）：「客戶是否要求真人客服？」
> - `is_repeat_contact`（是否重複聯絡）：「客戶之前是否曾就此事聯絡過客服？」
>   - `true`（是）：提到先前的嘗試、工單，或說自己之前問過
>   - `false`（否）：沒有任何先前聯絡的跡象
>
> 原網頁在此範例下方有「Try it in the Playground →」連結，可在 [Playground](https://console.typesafe.ai/playground) 中直接試用。

問題 ID 由你自己決定，這裡是 `is_human_escalation` 與 `is_repeat_contact`。這些 ID 不會傳送給模型；每個答案會在同一個 ID 底下回傳。第一個問題只靠 `instructions`；第二個問題另外加上 `criteria`，說明什麼算「是」、什麼算「否」。

使用 [Python SDK](https://docs.typesafe.ai/sdk/python.md) 時，同樣的問題寫成 `Noul` 物件：

```python
from typesafe_sdk import Noul, NoulCriteria, TypeSafeClient

with TypeSafeClient() as client:
    response = client.system_one(
        model="jev-latest",
        state="I have asked three times now. Can I please just talk to a real person?",
        questions={
            "is_human_escalation": Noul(
                instructions="Is the customer asking for a human agent?",
            ),
            "is_repeat_contact": Noul(
                instructions="Has the customer contacted support about this before?",
                criteria=NoulCriteria(
                    true="Mentions a prior attempt, ticket, or that they have asked before",
                    false="No sign of any previous contact",
                ),
            ),
        },
    )

    print(response.answers["is_human_escalation"].noul)
    print(response.answers["is_repeat_contact"].noul)
```

`system_one` 方法與 `https://api.typesafe.ai/v1/systemone` 端點，都是以 TypeSafe 的 AI 模型 [System One](../concepts/system-one.zh.md) 命名的。[如何使用 TypeSafe 建置](../concepts/how-to-build-with-system-one.zh.md)說明了應該在程式碼的哪裡使用它。

如果你使用程式開發 Agent，請先安裝 [TypeSafe Agent 技能](https://docs.typesafe.ai/agent-skill.md#installation)，讓它知道請求與回應的格式。

> **注意：** `instructions` 可以是字串、物件或陣列。先從字串開始。當問題需要搭配資料時（例如要拿一筆紀錄和狀態比對），或問題的一部分是由你的程式碼產生時，就使用物件。[在問題中使用結構](../concepts/how-to-build-with-system-one.zh.md#在問題中使用結構)說明了結構何時有幫助，[下方的範例](#結構化的指示)則示範由程式碼產生的問題。

## 回應結構

回應中的 `answers` 對每個問題各有一個項目，放在請求時使用的 ID 底下：

```json
{
  "model": "jev-1.13.0",
  "answers": {
    "is_human_escalation": {
      "type": "noul",
      "noul": 0.99
    },
    "is_repeat_contact": {
      "type": "noul",
      "noul": 0.93
    }
  },
  "usage": {
    "input_tokens": 360,
    "output_tokens": 39
  }
}
```

這裡兩個答案都接近 1。客戶說了「跟真人說話」，所以 `is_human_escalation` 是 0.99。「我已經問了三次了」符合 `is_repeat_contact` 的 `true` 描述，所以是 0.93。

## 解讀 Noul

這個數字**同時是答案，也是確定程度**。接近 1 是強烈的「是」，接近 0 是強烈的「否」，接近 0.5 則代表模型認為「是」與「否」的機率相近。

下表是 `jev-1.13.0` 針對 `is_human_escalation` 問題，對不同客戶訊息的實際回答紀錄：

| 狀態 | `noul` |
| --- | --- |
| Thanks, that fixed it!<br/>（謝謝，這樣就解決了！） | 0.02 |
| How do I reset my password?<br/>（我要怎麼重設密碼？） | 0.07 |
| I need this sorted today, whatever it takes.<br/>（不管怎樣，我今天就需要把這件事解決。） | 0.26 |
| Are you a bot?<br/>（你是機器人嗎？） | 0.40 |
| Is there any way to speak to someone about my invoice?<br/>（有沒有辦法找個人談談我的發票問題？） | 0.84 |
| I have asked three times now. Can I please just talk to a real person?<br/>（我已經問了三次了。可以拜託讓我直接跟真人說話嗎？） | 0.99 |

前兩個和最後兩個都很明確。「我今天就需要把這件事解決」很緊急，但從頭到尾沒有要求找人，得到 0.26。「你是機器人嗎？」暗示想找真人但沒有明說，模型的判斷幾乎對半，給了 0.40。這兩種訊息，正是需要在程式碼中依門檻值做決定的類型。

和 [Choice](choice.zh.md) 或 [Score](score.zh.md) 不同，Noul **沒有另外的 `confidence` 值**。Noul 的機率分布只有兩種結果：是與否，所以單一個 `noul` 值就能完整描述它。Choice 或 Score 則把機率分散在多個選項或等級上，`confidence` 是用來概括這種分散程度的。

最常見的做法，是在程式碼中用門檻把 `noul` 轉成布林值：

```python
wants_human = response.answers["is_human_escalation"].noul > 0.9

if wants_human:
    route_to_agent(ticket)
else:
    route_to_bot(ticket)
```

> 中文對照：`noul` > 0.9 就視為「想找真人」，轉給真人客服（`route_to_agent`）；否則轉給機器人（`route_to_bot`）。

門檻要設在哪裡，取決於犯錯的代價：

- 當「是」和「否」都一樣容易處理時，用 0.5。
- 當「誤判為是」而採取行動的代價很高時（例如呼叫值班人員或發放退款），就提高門檻。
- 當「漏掉真正的是」的代價很高時（例如沒有標記出安全問題），就降低門檻。
- 落在中間的值可以交給人處理，而不走任何一條程式路徑。這和[信心度](../confidence.zh.md#在程式碼中運用信心度的三條路徑)頁面為 Choice 與 Score 答案描述的三分法相同。

Noul 的值介於 0 到 1，但它**不是**你所問事物的量表，而是「答案為是」的機率。如果問題其實是在問程度，這個值並不能衡量程度。下面用「這位應徵者的 Python 能力強嗎？」詢問四位應徵者，並和一個有四個等級（沒有經驗、略有接觸、在工作中經常使用、深厚專業）的 [Score](score.zh.md) 並列比較：

| 應徵者 | Noul：「這位應徵者的 Python 能力強嗎？」 | Score：「這位應徵者有多少 Python 經驗？」 |
| --- | --- | --- |
| My experience is in Java and Go. I have not used Python.<br/>（我的經驗在 Java 和 Go。我沒用過 Python。） | 0.03 | 0.0（沒有經驗） |
| I have used Python occasionally for small scripts alongside my main Java work.<br/>（除了主要的 Java 工作外，我偶爾用 Python 寫小腳本。） | 0.14 | 1.0（略有接觸） |
| I used Python every day for two years in my last job, mostly data pipelines.<br/>（在上一份工作中，我兩年來每天都用 Python，主要是資料管線。） | 0.81 | 2.05（在工作中經常使用） |
| I have written Python daily for eight years, including maintaining a large Django codebase.<br/>（我八年來每天都寫 Python，包括維護一個大型 Django 程式碼庫。） | 0.92 | 2.89（深厚專業） |

Noul 判斷的是單一命題「強」，而這些值是這個命題成立的可能性。你可以在程式碼中為 0 到 1 的範圍自訂區間（例如 0.3 到 0.7 代表「有些經驗」），但模型看不到這些區間，所以答案中沒有任何部分是依據它們判斷的。中間的值可能代表「中等經驗」，也可能代表「情況不明」；而且應徵者之間的間距也不是你所決定的。Score 則會各自判斷每一個等級描述，所以每位應徵者都落在你寫的某個等級上或附近，回傳的機率也顯示模型如何在各等級之間分配它的判斷。如果你不同意結果，就改寫某個等級的措辭再執行一次。[選擇問題類型](../primitives.zh.md#選擇問題類型)說明了兩者的區別。

## 撰寫 Noul 問題

**每個 Noul 只問一個是非題。** 如果一個問題有兩個條件，例如「客戶是否很生氣**而且**要求退款？」，模型就必須同時判斷兩件事，這個值的意義也會變弱。請改成兩個 Noul，再在程式碼中組合。

**措辭要讓「高值代表是」。**「這則訊息是否包含個人資料？」很清楚；「這則訊息是否不含個人資料？」則把意思反過來了，之後讀取它的程式碼很容易搞反。

**敘述句和問句一樣有效。** 對於「客戶正在要求退款」這個敘述，接近 1 的值代表這個敘述為真。請用你自己的資料試試兩種寫法，看哪一種效果比較好。

**讓「是」與「否」的界線明確。**「這位應徵者有沒有**任何** Python 經驗？」效果很好，因為「任何」沒有留下模糊地帶。當界線比較微妙時，就加上帶有 `true` 與 `false` 描述的 `criteria`，就像上面的 `is_repeat_contact` 問題那樣。大多數 Noul 只靠指示就夠了，所以請分別試試有無 `criteria` 的版本，保留在你的文件上表現較好的那一個。

## 良好做法：每次呼叫提出不只一個問題

面對一份條件檢查清單時，請在一次請求中提出多個 Noul 問題：每個條件一個問題，再由程式碼決定這些結果組合起來代表什麼。問題會平行評估，所以增加 Noul 幾乎不會影響回應時間。[一起提出多個問題](../primitives.zh.md#一起提出多個問題)有更詳細的說明。

## 在程式碼中處理多個 Noul 答案

上面那個有兩個問題的請求，已經提供了足夠的資訊讓程式碼路由這則訊息。下面的範例在客戶要求找真人時轉給真人，並在客戶之前聯絡過時提高優先順序。任一問題的值若落在中間，就交給審核人員，而不走任何一條程式路徑：

```python
from typesafe_sdk import Noul, NoulCriteria, TypeSafeClient

SUPPORT_QUESTIONS = {
    "is_human_escalation": Noul(
        instructions="Is the customer asking for a human agent?",
    ),
    "is_repeat_contact": Noul(
        instructions="Has the customer contacted support about this before?",
        criteria=NoulCriteria(
            true="Mentions a prior attempt, ticket, or that they have asked before",
            false="No sign of any previous contact",
        ),
    ),
}

YES = 0.8
NO = 0.2


def route(message: str) -> None:
    with TypeSafeClient() as client:
        response = client.system_one(
            model="jev-latest",
            state=message,
            questions=SUPPORT_QUESTIONS,
        )
    answers = response.answers

    wants_human = answers["is_human_escalation"].noul
    repeat = answers["is_repeat_contact"].noul

    if NO < wants_human < YES or NO < repeat < YES:
        # The model isn't sure either way. Let a person decide.
        send_to_review(message)
        return

    priority = "high" if repeat > YES else "normal"
    if wants_human > YES:
        route_to_agent(message, priority=priority)
    else:
        route_to_bot(message, priority=priority)
```

> 中文對照（程式邏輯）：
> - `YES = 0.8`、`NO = 0.2`：高於 0.8 視為「是」，低於 0.2 視為「否」。
> - 任一個值落在 0.2 到 0.8 之間：「模型兩邊都不確定。交給人決定。」→ 送交審核（`send_to_review`），並結束。
> - 重複聯絡 > 0.8 → 優先順序為 `high`（高），否則為 `normal`（一般）。
> - 要求找真人 > 0.8 → 轉給真人客服（`route_to_agent`）；否則轉給機器人（`route_to_bot`）。兩者都帶上優先順序。

以上面那則訊息來說，`is_human_escalation` 的值是 0.99、`is_repeat_contact` 是 0.93，所以程式碼會以高優先順序把它轉給真人客服。「我要怎麼重設密碼？」這則訊息在兩個問題上都是 0.07，會被轉給機器人。

門檻存在於你的程式碼中。如果審核人員看到太多訊息，就縮小 `NO` 與 `YES` 之間的區間；如果有太多錯誤的路由漏網，就把區間放寬。如果之後需要知道訊息是否提到付款，或是否包含個人資料，只要在 `SUPPORT_QUESTIONS` 再加一個 Noul 即可，請求次數仍然只有一次。

## 結構化的指示

指示可以是物件而不是字串：問題放在一個欄位，補充資料放在其他欄位。[在問題中使用結構](../concepts/how-to-build-with-system-one.zh.md#在問題中使用結構)說明了這在什麼時候有幫助。這裡用它來處理由程式碼產生的問題：把一份剛收到的履歷，和應徵者資料庫中「可能是同一個人」的紀錄做比對。每筆紀錄原封不動地放進 `potential_duplicate` 欄位，每筆紀錄的 `question` 都相同，所有紀錄在一次請求中檢查完畢。由程式碼產生的問題鍵中包含每筆紀錄的資料庫 ID：

**request**

```json
{
  "state": {
    "resume": {
      "name": "John Smith",
      "location": "Oakland, CA",
      "summary": "Backend engineer with eight years of Python and Go experience.",
      "experience": [
        { "employer": "Google", "title": "Senior Backend Engineer", "years": "2021-2025" },
        { "employer": "Microsoft", "title": "Software Engineer", "years": "2017-2021" }
      ]
    }
  },
  "model": "jev-latest",
  "questions": {
    "same_as_record_18": {
      "type": "noul",
      "instructions": {
        "potential_duplicate": { "name": "Jon Smith", "location": "Oakland, CA", "last_employer": "Google" },
        "question": "Is the resume for the same person as `potential_duplicate`?"
      }
    },
    "same_as_record_42": {
      "type": "noul",
      "instructions": {
        "potential_duplicate": { "name": "John Smith", "location": "Austin, TX", "last_employer": "Lone Star Freight" },
        "question": "Is the resume for the same person as `potential_duplicate`?"
      }
    },
    "same_as_record_77": {
      "type": "noul",
      "instructions": {
        "potential_duplicate": { "name": "John Smithers", "location": "Oakland, CA", "last_employer": "Bay Health Clinic" },
        "question": "Is the resume for the same person as `potential_duplicate`?"
      }
    }
  }
}
```

> 中文對照：
> - 狀態（履歷）：John Smith，加州奧克蘭；摘要「有八年 Python 與 Go 經驗的後端工程師」；經歷：Google 資深後端工程師（2021–2025）、Microsoft 軟體工程師（2017–2021）。
> - 三個問題的 `question` 都是：「這份履歷和 `potential_duplicate` 是同一個人嗎？」
> - `potential_duplicate`（可能重複的紀錄）：
>   - 紀錄 18：Jon Smith，加州奧克蘭，最近雇主 Google
>   - 紀錄 42：John Smith，德州奧斯汀，最近雇主 Lone Star Freight
>   - 紀錄 77：John Smithers，加州奧克蘭，最近雇主 Bay Health Clinic
>
> 原網頁在此範例下方有「Try it in the Playground →」連結，可在 [Playground](https://console.typesafe.ai/playground) 中直接試用。

回應：

```json
{
  "model": "jev-1.13.0",
  "answers": {
    "same_as_record_18": {
      "type": "noul",
      "noul": 0.74
    },
    "same_as_record_42": {
      "type": "noul",
      "noul": 0.09
    },
    "same_as_record_77": {
      "type": "noul",
      "noul": 0.08
    }
  },
  "usage": {
    "input_tokens": 535,
    "output_tokens": 58
  }
}
```

每個答案都是「這份履歷屬於該筆紀錄中的那個人」的機率：

- 紀錄 18 的名字拼法不同，但地點和雇主都相符，得到 0.74。
- 紀錄 42 名字相同，但城市和雇主都不同，得到 0.09。
- 紀錄 77 名字相似、地點相同，但雇主不同，得到 0.08。

請在程式碼中對每個值套用門檻，做法如[在程式碼中處理多個 Noul 答案](#在程式碼中處理多個-noul-答案)所示，並把中間的值交給人處理。

使用 Python SDK 時，問題是由應徵者紀錄產生的。問題文字固定不變，變的是紀錄：

```python
from typesafe_sdk import Noul, TypeSafeClient

SAME_PERSON = "Is the resume for the same person as `potential_duplicate`?"


def duplicate_questions(candidates: list[dict]) -> dict[str, Noul]:
    """One Noul per candidate record, all asking the same question."""
    return {
        f"same_as_record_{candidate['id']}": Noul(
            instructions={
                "potential_duplicate": {
                    "name": candidate["name"],
                    "location": candidate["location"],
                    "last_employer": candidate["last_employer"],
                },
                "question": SAME_PERSON,
            },
        )
        for candidate in candidates
    }


def find_duplicates(resume: dict, candidates: list[dict]) -> list[str]:
    with TypeSafeClient() as client:
        response = client.system_one(
            model="jev-latest",
            state={"resume": resume},
            questions=duplicate_questions(candidates),
        )
    return [
        question_id
        for question_id, answer in response.answers.items()
        if answer.noul > 0.7
    ]
```

> 中文對照：
> - `duplicate_questions` 的說明：「每筆候選紀錄一個 Noul，全部問同一個問題。」
> - `find_duplicates` 回傳所有 `noul` > 0.7 的問題 ID，也就是可能重複的紀錄。

[結構化資料擷取串接操作手冊](https://docs.typesafe.ai/cookbooks/sde_cascade.md)使用結構化的指示來驗證擷取出來的紀錄。每個欄位都套用同一組問題。每個問題的 `instructions` 物件中，問題文字放在 `main_question` 屬性；另外還有 `field_spec` 與 `extracted_field` 屬性，會隨每個欄位而改變。

## 操作手冊中的 Noul

看看我們的操作手冊，了解使用 Noul 問題的應用：

* [平行提問](https://docs.typesafe.ai/cookbooks/parallel_questions.md)：在一次請求中，對一篇文章執行 13 個問題的法規檢查清單。
* [自我一致性：Noul](https://docs.typesafe.ai/cookbooks/consistency_noul_cookbook.md)：依據一份 15 個問題的評分準則替保險理賠案評分，並衡量這些值在多次執行間有多穩定。
* [重新排序](https://docs.typesafe.ai/cookbooks/rerank_typesafe.md)：直接使用機率本身，而不是門檻。每組「查詢—候選」配對一個 Noul，再依這個值替候選排序。
* [逐行搜尋](https://docs.typesafe.ai/cookbooks/semantic_find.md)：搭配一個找出符合行的 Choice，以及一個檢查文件中是否根本有答案的 Noul。
* [結構還原](https://docs.typesafe.ai/cookbooks/autoformat.md)：每一對相鄰的行各問一個 Noul，判斷換行是否切斷了句子，藉此從純文字重建段落。
