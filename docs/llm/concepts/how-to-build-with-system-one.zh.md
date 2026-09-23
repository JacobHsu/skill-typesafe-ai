> 本文件是 [`how-to-build-with-system-one.md`](how-to-build-with-system-one.md)（來源：<https://docs.typesafe.ai/concepts/how-to-build-with-system-one.md>，下載於 2026-09-23）的繁體中文翻譯，供人類閱讀參考。
> 若內容有出入，以線上英文版本為準。程式碼保持原文，範例中的英文字串另附中文對照。
> 原文開頭有一段網站用來產生 Playground 連結的 JavaScript 元件（`TypesafeExample`），與內容無關，本譯文已省略，範例改寫成 JSON 程式碼區塊。原網頁中可展開的範例區塊（Accordion）在此直接展開。

> ## 文件索引
> 完整的文件索引請見：https://docs.typesafe.ai/llms.txt（中文版：[llms.zh.md](../../llms.zh.md)）
> 在深入探索之前，可先用這份索引了解所有可用的頁面。

# 如何使用 TypeSafe 建置

> 設計 AI 驅動的軟體時，讓程式碼保有掌控權，只把範圍明確、結構化的決策交給 System One。

System One 是 TypeSafe 用來打造 **AI 驅動軟體**的模型，而不是用來打造 Agent。它不會產生程式碼，也不會自行選擇下一步動作。它提供可以嵌入軟體的 AI 原語，讓程式碼保有掌控權，而模型負責處理非結構化資料上的常識判斷。

> **摘要：** 建置一個一般的軟體工作流程，只在需要 AI 的地方插入 System One。
>
> * 控制流程、確定性規則與副作用都留在程式碼中。
> * 把廣泛的判斷拆解成範圍明確、具型別的問題，並附上明確的指示與準則。
> * 只給每個問題它所需要的上下文。
> * 用機率與信心度來決定：直接行動、請求審核，或升級處理。
> * 把彼此獨立的問題一起提出，再在程式碼中組合它們的答案。

## 三種軟體架構

TypeSafe 是為了打造 **AI 驅動的軟體**而設計的：由程式碼掌控工作流程，由 AI 處理範圍明確的結構化決策。

**傳統軟體**

傳統程式碼是由簡單的軟體原語構成的複雜決策樹。因為每個原語都很可靠，開發者可以把它們組合成更高層次的抽象。

**LLM Agent**

Agent 處理指令，並自行選擇下一步。當有人在旁監看整個過程時，這種方式運作良好；但每一次迴圈，都是另一個可能脫軌的機會。

**AI 驅動的軟體**

程式碼處理確定性的工作，並掌控控制流程。只有在系統需要「可程式化的常識」或需要解讀非結構化資料時，模型才會出現。每個 AI 任務都保持原子化且受到限制。

![傳統軟體、Agent 與 AI 驅動的軟體，以三種不同的系統架構呈現。](https://mintcdn.com/ts-docs/aFVnpmCIX68NpsV1/images/how-to-build-with-typesafe/software-architectures-light.webp?fit=max&auto=format&n=aFVnpmCIX68NpsV1&q=85&s=35c7622176190d1b1f19dc712f2fbf11)

## System One 為何可以組合

- **結構化（Structured）**：System One 在設計上就是型別安全的。決策與機率都符合你的程式碼所預期的結構化軟體型別與 JSON schema，所以程式碼永遠不需要從生成的文字中撈出數值。
- **平行（Parallel）**：問題彼此獨立、平行評估。一個原語的結果不會成為改變另一個原語結果的隱藏上下文。
- **可比較（Comparable）**：輸出可以排序，並能驅動聰明的 `if` 敘述、門檻與比較。
- **快速（Fast）**：大多數查詢約在 100 毫秒內完成。System One 快到足以用在即時的請求路徑與使用者介面中。
- **經過校準的信心度（Calibrated confidence）**：[RLCD](https://docs.typesafe.ai/introduction/machine-learning-primer.md) 透過經過校準的機率來傳達不確定性，而不會傾向過度自信。
- **自我一致（Self-consistent）**：System One 的設計目標是在重複評估時回傳穩定的答案。請見[自我一致性操作手冊](https://docs.typesafe.ai/cookbooks/consistency_noul_cookbook.md)。

因為每個輸出都被限制在所提供的選項之內，模型回傳的是這些選項上完整的機率分布，而不會憑空捏造 schema 之外的值。TypeSafe 的目標是達到超過 100 倍的「智慧對速度與成本比」；背後的賭注是：更便宜的智慧將創造出多得多的需求。

## 設計 System One 工作流程

以下八個步驟依序說明設計方式：

1. [能用程式碼就用程式碼](#能用程式碼就用程式碼)
2. [拆解輸入狀態](#拆解輸入狀態)
3. [在輸入狀態中使用結構](#在輸入狀態中使用結構)
4. [拆解問題](#拆解問題)
5. [在問題中使用結構](#在問題中使用結構)
6. [提出大量問題](#提出大量問題)
7. [在程式碼中組合問題的輸出（或送入傳統機器學習模型）](#在程式碼中組合問題的輸出或送入傳統機器學習模型)
8. [依不確定性路由](#依不確定性路由)

### 能用程式碼就用程式碼

確定性的工作留在程式碼中，它既可靠又便宜。當軟體工作流程就能表達同樣的行為時，避免使用 Agent 的 `while` 迴圈。

**範例：確定性規則留在程式碼中**

```python
days_overdue = (today - invoice.due_date).days

if days_overdue > 30:
    route_to_collections(invoice)
```

> 中文對照：計算發票逾期天數；逾期超過 30 天就轉給催收（`route_to_collections`）。這種規則不需要模型。

瀏覽 [System One 模式](https://docs.typesafe.ai/patterns.md)，了解以有界方式把模型決策與程式碼組合起來的做法。

### 拆解輸入狀態

只放入與目前問題相關的上下文。這能幫助模型避免分心與上下文劣化（context rot）。當目前的資訊可以從你自己的知識庫取得時，不要依賴儲存在模型權重中的知識。

**範例：只送出相關的上下文**

```json
{
  "state": {
    "ticket_message": "My flight was cancelled. Can I get a refund?",
    "refund_policy": "Cancelled flights are eligible for a full refund."
  },
  "model": "jev-latest",
  "questions": {
    "policy_supports_refund": {
      "type": "noul",
      "instructions": "Does the refund policy support the refund requested in the ticket?"
    }
  }
}
```

> 中文對照：
> - `ticket_message`：「我的航班被取消了。我可以退款嗎？」
> - `refund_policy`：「被取消的航班符合全額退款資格。」
> - `policy_supports_refund`（Noul）：「退款政策是否支持工單中所要求的退款？」
>
> 重點：只放工單訊息和相關的退款政策，不放整份客戶資料或其他無關政策。

### 在輸入狀態中使用結構

`state` 與 `questions` 欄位都可以使用巢狀 JSON。當指向特定值能消除歧義時，就讓問題指向它，並在問題中用反引號把每個路徑包起來。

**範例：參照巢狀的值**

使用反引號包住的「點號加索引」路徑，讓問題指向特定的巢狀值，例如 `support.tickets[0].message`。

```json
{
  "state": {
    "support": {
      "tickets": [
        { "message": "I was charged twice for order A-104." },
        { "message": "How do I reset my password?" }
      ]
    },
    "commerce": {
      "orders": [
        {
          "id": "A-104",
          "charges": [
            { "amount_usd": 49, "status": "captured" },
            { "amount_usd": 49, "status": "captured" }
          ]
        }
      ]
    },
    "account": {
      "security": {
        "password_reset": "Email a reset link to the address on file."
      }
    }
  },
  "model": "jev-latest",
  "questions": {
    "duplicate_charge": {
      "type": "noul",
      "instructions": "Do `support.tickets[0].message` and `commerce.orders[0].charges` indicate a duplicate charge?"
    },
    "password_reset_supported": {
      "type": "noul",
      "instructions": "Can `account.security.password_reset` resolve the request in `support.tickets[1].message`?"
    }
  }
}
```

> 中文對照：
> - 工單 0：「訂單 A-104 被扣款了兩次。」；工單 1：「我要怎麼重設密碼？」
> - `commerce.orders[0]`：訂單 A-104，兩筆各 49 美元、狀態為 `captured`（已請款）的扣款
> - `account.security.password_reset`：「寄送重設連結到檔案上登記的電子郵件地址。」
> - `duplicate_charge`（重複扣款）：「`support.tickets[0].message` 與 `commerce.orders[0].charges` 是否顯示有重複扣款？」
> - `password_reset_supported`（可否重設密碼）：「`account.security.password_reset` 能否解決 `support.tickets[1].message` 中的請求？」

### 拆解問題

盡可能提出最明確、範圍最窄、最具體、最原子化的問題。把複雜或定義不清的問題，拆成各自只評估一個屬性的獨立問題。

> **這大概是本指南中最重要的概念。** 廣泛的問題會把好幾個判斷藏在同一個答案背後；原子化的問題則把這些判斷攤開，讓你能在程式碼中檢查、調整並組合它們。

**範例：拆解垃圾郵件偵測**

兩個範例使用同一份狀態：

```json
{
  "message": {
    "sender": {
      "display_name": "Acme Payroll",
      "email": "rewards@claim-bonus.example"
    },
    "subject": "Urgent: claim your employee bonus",
    "body": "You have been selected for a $1,000 bonus. Confirm your payroll password today to receive it.",
    "links": [
      {
        "text": "Claim bonus",
        "url": "http://claim-bonus.example/acme"
      }
    ]
  }
}
```

> 中文對照：寄件者顯示名稱「Acme 薪資部」，但電子郵件是 `rewards@claim-bonus.example`；主旨「緊急：領取你的員工獎金」；內文「你已獲選領取 1,000 美元獎金。請於今天確認你的薪資系統密碼以領取。」；連結文字「領取獎金」，網址 `http://claim-bonus.example/acme`。

**一個廣泛的問題（不好）**

```json
{
  "is_spam": {
    "type": "noul",
    "instructions": "Is `message` spam?"
  }
}
```

> 中文對照：`is_spam`：「`message` 是垃圾郵件嗎？」

**拆解後的問題（好）**

```json
{
  "requests_credentials": {
    "type": "noul",
    "instructions": "Does `message.body` ask the recipient to provide a password or other login credential?"
  },
  "offers_unexpected_reward": {
    "type": "noul",
    "instructions": "Does `message.body` claim the recipient received an unexpected prize, payment, or reward?"
  },
  "creates_time_pressure": {
    "type": "noul",
    "instructions": "Does `message.subject` or `message.body` pressure the recipient to act quickly?"
  },
  "sender_identity_mismatch": {
    "type": "noul",
    "instructions": "Does the organization named in `message.sender.display_name` conflict with the domain in `message.sender.email`?"
  },
  "link_domain_mismatch": {
    "type": "noul",
    "instructions": "Does the domain in `message.links[0].url` conflict with the organization named in `message.sender.display_name`?"
  },
  "disguises_link_destination": {
    "type": "noul",
    "instructions": "Does `message.links[0].text` conceal or misrepresent the destination in `message.links[0].url`?"
  }
}
```

> 中文對照：
> - `requests_credentials`（索取帳密）：「`message.body` 是否要求收件者提供密碼或其他登入憑證？」
> - `offers_unexpected_reward`（意外獎勵）：「`message.body` 是否聲稱收件者獲得了意料之外的獎品、款項或獎勵？」
> - `creates_time_pressure`（製造時間壓力）：「`message.subject` 或 `message.body` 是否催促收件者盡快行動？」
> - `sender_identity_mismatch`（寄件者身分不符）：「`message.sender.display_name` 中的組織名稱，是否與 `message.sender.email` 的網域相衝突？」
> - `link_domain_mismatch`（連結網域不符）：「`message.links[0].url` 的網域，是否與 `message.sender.display_name` 中的組織名稱相衝突？」
> - `disguises_link_destination`（偽裝連結目的地）：「`message.links[0].text` 是否隱藏或誤導了 `message.links[0].url` 的實際目的地？」

**範例：驗證工具呼叫的追蹤紀錄**

兩個範例使用同一份狀態：

```json
{
  "request": {
    "text": "What's the weather in Seattle tomorrow in Fahrenheit?",
    "location": "Seattle, WA",
    "date": "2026-09-03",
    "unit": "fahrenheit"
  },
  "available_tools": {
    "geocode_city": {
      "description": "Resolve a city to latitude and longitude.",
      "parameters": { "city": "string" }
    },
    "get_weather": {
      "description": "Get the forecast for coordinates and a date.",
      "parameters": {
        "latitude": "number",
        "longitude": "number",
        "date": "YYYY-MM-DD",
        "unit": ["fahrenheit", "celsius"]
      }
    }
  },
  "trace": {
    "tool_calls": [
      {
        "id": "call_1",
        "name": "geocode_city",
        "arguments": { "city": "Seattle, WA" }
      },
      {
        "id": "call_2",
        "name": "get_weather",
        "arguments": {
          "latitude": 47.6062,
          "longitude": -122.3321,
          "date": "2026-09-03",
          "unit": "celsius"
        }
      }
    ],
    "tool_results": [
      {
        "tool_call_id": "call_1",
        "output": { "latitude": 47.6062, "longitude": -122.3321 }
      }
    ]
  }
}
```

> 中文對照：
> - `request`：使用者問「西雅圖明天的天氣如何？用華氏溫度。」，地點西雅圖、日期 2026-09-03、單位 `fahrenheit`（華氏）。
> - `available_tools`：`geocode_city`（把城市解析成經緯度）、`get_weather`（取得某座標與日期的天氣預報，單位可為華氏或攝氏）。
> - `trace`：Agent 先呼叫 `geocode_city` 取得西雅圖的經緯度，再呼叫 `get_weather`，但單位傳的是 `celsius`（攝氏）。
>
> （譯註：這份追蹤紀錄故意藏了一個錯誤：使用者要華氏，工具呼叫卻用了攝氏。）

**一個廣泛的問題（不好）**

```json
{
  "tool_calls_are_correct": {
    "type": "noul",
    "instructions": "Is `trace.tool_calls` correct for `request` and `available_tools`?"
  }
}
```

> 中文對照：`tool_calls_are_correct`：「對於 `request` 與 `available_tools` 而言，`trace.tool_calls` 正確嗎？」

**拆解後的問題（好）**

```json
{
  "geocode_tool_is_relevant": {
    "type": "noul",
    "instructions": "Is `trace.tool_calls[0].name` an appropriate tool for resolving `request.location`?"
  },
  "geocode_location_matches": {
    "type": "noul",
    "instructions": "Does `trace.tool_calls[0].arguments.city` match `request.location`?"
  },
  "geocode_arguments_match_schema": {
    "type": "noul",
    "instructions": "Does `trace.tool_calls[0].arguments` conform to `available_tools.geocode_city.parameters`?"
  },
  "geocode_result_matches_call": {
    "type": "noul",
    "instructions": "Does `trace.tool_results[0].tool_call_id` match `trace.tool_calls[0].id`?"
  },
  "weather_tool_is_relevant": {
    "type": "noul",
    "instructions": "Is `trace.tool_calls[1].name` an appropriate tool for answering `request.text`?"
  },
  "weather_arguments_match_schema": {
    "type": "noul",
    "instructions": "Does `trace.tool_calls[1].arguments` conform to `available_tools.get_weather.parameters`?"
  },
  "weather_uses_geocoded_coordinates": {
    "type": "noul",
    "instructions": "Do the coordinates in `trace.tool_calls[1].arguments` match those in `trace.tool_results[0].output`?"
  },
  "weather_date_matches": {
    "type": "noul",
    "instructions": "Does `trace.tool_calls[1].arguments.date` match `request.date`?"
  },
  "weather_unit_matches": {
    "type": "noul",
    "instructions": "Does `trace.tool_calls[1].arguments.unit` match `request.unit`?"
  }
}
```

> 中文對照：
> - `geocode_tool_is_relevant`：「`trace.tool_calls[0].name` 是用來解析 `request.location` 的適當工具嗎？」
> - `geocode_location_matches`：「`trace.tool_calls[0].arguments.city` 與 `request.location` 相符嗎？」
> - `geocode_arguments_match_schema`：「`trace.tool_calls[0].arguments` 符合 `available_tools.geocode_city.parameters` 的格式嗎？」
> - `geocode_result_matches_call`：「`trace.tool_results[0].tool_call_id` 與 `trace.tool_calls[0].id` 相符嗎？」
> - `weather_tool_is_relevant`：「`trace.tool_calls[1].name` 是用來回答 `request.text` 的適當工具嗎？」
> - `weather_arguments_match_schema`：「`trace.tool_calls[1].arguments` 符合 `available_tools.get_weather.parameters` 的格式嗎？」
> - `weather_uses_geocoded_coordinates`：「`trace.tool_calls[1].arguments` 中的座標，與 `trace.tool_results[0].output` 中的座標相符嗎？」
> - `weather_date_matches`：「`trace.tool_calls[1].arguments.date` 與 `request.date` 相符嗎？」
> - `weather_unit_matches`：「`trace.tool_calls[1].arguments.unit` 與 `request.unit` 相符嗎？」
>
> （譯註：拆解後，錯誤會精準地出現在 `weather_unit_matches` 這一題；廣泛的問題則只能給出一個籠統的機率，看不出錯在哪裡。）

### 在問題中使用結構

問題要保持簡短。`instructions` 與 `criteria` 通常是字串；對於簡短、明確的問題，字串就夠了。它們也可以是物件或陣列：把問題放在一個欄位，把引導這個問題的資料放在其他欄位。

在以下情況中，結構會有幫助：

* **問題需要上下文或範例。** 一長句背景資訊或一串範例輸入，應該放在問題旁邊的具名欄位中；這樣你的程式碼就能增加或替換它們，而不必改寫問題本身。
* **問題有一部分來自你的程式碼。** 當某個值來自資料庫時，把它放在獨立的欄位中，而不是把它拼接進字串範本裡。
* **多個問題的指示很相似。** 一次請求只有一份狀態，但可以包含多個問題。加入補充資料能幫助這些問題彼此區分。

**範例：參照來自程式碼的紀錄**

這個 Noul 會拿狀態中的履歷，和應徵者資料庫中的一筆紀錄做比對。這筆紀錄原封不動地放進 `potential_duplicate`，問題則用名稱參照它。

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
        "potential_duplicate": { "name": "John Smith", "location": "Oakland, California", "last_employer": "Google" },
        "question": "Is the resume for the same person as `potential_duplicate`?"
      }
    }
  }
}
```

> 中文對照：`question`＝「這份履歷和 `potential_duplicate`（可能重複的紀錄）是同一個人嗎？」完整說明請見 [Noul](../primitives/noul.zh.md#結構化的指示) 頁面的同一個範例。

來自程式碼的 `potential_duplicate` 資料可能隨時間改變，而 `question` 則用反引號來參照它。

`criteria` 裡的描述也可以是物件。對 Choice 而言，每個選項的描述可以是一個物件，說明這個選項涵蓋什麼、哪些內容屬於別的選項，以及幾個範例。所有選項使用相同的欄位名稱，讓模型能直接比較它們。

**範例：定義對比式的 Choice 準則**

```json
{
  "state": "How many disposable virtual cards can I make per day?",
  "model": "jev-latest",
  "questions": {
    "card_help_topic": {
      "type": "choice",
      "instructions": {
        "question": "Which disposable virtual card topic is the user asking about?",
        "focus": "Classify the information the user wants."
      },
      "criteria": {
        "get_disposable_virtual_card": {
          "what": "Purpose, eligibility, or setup",
          "not_for": "Quantity, transaction, or merchant restrictions",
          "examples": [
            "How can I get a disposable virtual card?",
            "What are disposable cards for?"
          ]
        },
        "disposable_card_limits": {
          "what": "Quantity, transaction, or merchant restrictions",
          "not_for": "Purpose, eligibility, or setup",
          "examples": [
            "How many disposable cards can I make per day?",
            "Where can I use a disposable card?"
          ]
        }
      }
    }
  }
}
```

> 中文對照：
> - 狀態：「我每天可以建立幾張一次性虛擬卡？」
> - `question`：「使用者詢問的是哪一個一次性虛擬卡相關主題？」；`focus`：「依使用者想要的資訊來分類。」
> - `get_disposable_virtual_card`（取得一次性虛擬卡）：
>   - `what`：用途、申請資格或設定方式
>   - `not_for`：數量、交易或商家限制
>   - `examples`：「我要怎麼取得一次性虛擬卡？」、「一次性卡片是做什麼用的？」
> - `disposable_card_limits`（一次性卡片的限制）：
>   - `what`：數量、交易或商家限制
>   - `not_for`：用途、申請資格或設定方式
>   - `examples`：「我每天可以建立幾張一次性卡片？」、「一次性卡片可以在哪裡使用？」

每種問題類型的頁面都有完整的範例：

* [Noul](../primitives/noul.zh.md#結構化的指示)：拿一份履歷和多筆應徵者紀錄比對，每筆紀錄一個問題，問題由程式碼產生。
* [Choice](../primitives/choice.zh.md#結構化的指示與準則)：描述兩個容易混淆的選項，各自說明涵蓋什麼、不適用於什麼，並附上範例。
* [Score](../primitives/score.zh.md#結構化的等級描述)：替每個等級提供描述與範例情境。

[結構化資料擷取串接操作手冊](https://docs.typesafe.ai/cookbooks/sde_cascade.md)示範了「共用措辭」的情況：對擷取出的紀錄中每一個欄位，都提出同一組問題。

簡短、明確的問題或準則可以維持字串形式。當結構能把原本會混在一起的指引分開時，再加入結構。所有接受結構的位置，請見[進階：結構](https://docs.typesafe.ai/primitives/advanced.md)。

### 提出大量問題

針對同一份狀態，在一次請求中提出大量範圍窄且彼此獨立的問題。這是使用 API 時，讓效果與「每一塊錢換到的智慧」最大化的方式：問題平行執行，程式碼可以組合它們的訊號，而不必增加一連串依序進行的模型往返。

請見[推測式扇出模式](../patterns/fan-out.zh.md)與[平行提問操作手冊](https://docs.typesafe.ai/cookbooks/parallel_questions.md)。

### 在程式碼中組合問題的輸出（或送入傳統機器學習模型）

用確定性規則或加權總和來組合彼此獨立的答案。若要用學習的方式組合，可以把這些機率當成特徵，送進下游的傳統機器學習模型。

**範例：用加權分數組合訊號**

```python
answers = response.answers

# Combine independent signals into one application-specific score.
quality = (
    0.4 * answers["answers_request"].noul
    + 0.4 * answers["citations_are_supported"].noul
    + 0.2 * (1 - answers["contradicts_context"].noul)
)
```

> 中文對照：
> - 註解：「把彼此獨立的訊號組合成一個應用程式專屬的分數。」
> - `answers_request`（有回答到請求）權重 0.4、`citations_are_supported`（引用有依據）權重 0.4、`contradicts_context`（與上下文矛盾）權重 0.2。
> - 注意最後一項用 `1 - …`：「與上下文矛盾」是負面訊號，所以反過來計算，機率越低分數越高。

[複合評分](../patterns/composite-scoring.zh.md)說明了如何在組合判斷的同時保留個別的判斷。如果你沒有可以訓練下游模型的標籤，可以用一組昂貴的推理模型來產生標籤；[AutoResearch 操作手冊](https://docs.typesafe.ai/cookbooks/autoresearch_feature_discovery.md)示範了如何用 System One 的輸出訓練傳統模型。

### 依不確定性路由

讓程式碼對有把握和沒把握的答案採取不同行動。把不確定的案例升級交給人工，或交給更昂貴的推理模型。在你的資料上畫出「信心度對準確率」的關係圖，藉此測試門檻。

**範例：依信心度路由**

```python
answer = response.answers["card_help_topic"]

if answer.confidence < 0.8:
    route_to_human_review(ticket)
else:
    route_to_handler(answer.choice, ticket)
```

> 中文對照：信心度低於 0.8 就送交人工審核（`route_to_human_review`）；否則依選中的主題轉給對應的處理程式（`route_to_handler`）。

選擇門檻、並讓門檻配合各個動作的風險，請見[信心度](../confidence.zh.md)與[信心度門檻路由](../patterns/confidence-routing.zh.md)。

> **提示：** 拆解問題並不需要更多次往返。針對同一份狀態的問題是平行執行的。

## 綜合應用

這個客服工單工作流程把確定性的工作留在程式碼中、只送出相關的結構化上下文、在一次請求中評估大量原子化的問題，並以明確的信心度關卡來組合答案。

**triage_ticket.py**

```python
from typesafe_sdk import Choice, Noul, NoulCriteria, Score, TypeSafeClient


def triage_ticket(ticket, customer):
    # Handle deterministic states without calling a model.
    if ticket["status"] == "closed":
        return "no_action"

    open_orders = [
        order for order in customer["orders"] if order["status"] != "delivered"
    ]

    # Include only the structured context needed by the questions below.
    state = {
        "ticket": {
            "message": ticket["message"],
            "sender": ticket["sender"],
            "links": ticket["links"],
        },
        "customer": {
            "plan": customer["plan"],
            "open_orders": open_orders,
        },
        "policy": {
            "sensitive_credentials": ["password", "security code", "API key"],
        },
    }

    # Ask structured, atomic questions together so they run in parallel.
    questions = {
        "topic": Choice(
            instructions={
                "question": "Which team should handle `ticket.message`?",
                "focus": "Classify the customer's primary request.",
            },
            criteria={
                "billing": {
                    "what": "Charges, invoices, refunds, or subscriptions",
                    "not_for": "Order tracking or account access",
                    "examples": ["I was charged twice", "Where is my refund?"],
                },
                "orders": {
                    "what": "Order status, delivery, cancellation, or returns",
                    "not_for": "Charges or account access",
                    "examples": ["Where is my order?", "Cancel my shipment"],
                },
                "account": {
                    "what": "Login, profile, permissions, or security",
                    "not_for": "Charges or order tracking",
                    "examples": ["Reset my password", "I cannot sign in"],
                },
            },
        ),
        "requests_credentials": Noul(
            instructions={
                "question": "Does the message request a sensitive credential?",
                "compare": [
                    "`ticket.message`",
                    "`policy.sensitive_credentials`",
                ],
                "focus": "Look for a request to disclose the credential itself.",
            },
            criteria=NoulCriteria(
                true={
                    "what": "Asks the recipient to disclose a listed credential",
                    "examples": [
                        "Reply with your password",
                        "Send us your API key",
                    ],
                },
                false={
                    "what": "Does not ask the recipient to disclose a credential",
                    "not_for": "A legitimate instruction to reset a credential",
                    "examples": ["Use this link to reset your password"],
                },
            ),
        ),
        "sender_identity_mismatch": Noul(
            instructions={
                "question": "Does the claimed sender identity conflict with its domain?",
                "compare": [
                    "`ticket.sender.display_name`",
                    "`ticket.sender.email`",
                ],
                "focus": "Compare the named organization with the email domain.",
            },
            criteria=NoulCriteria(
                true={
                    "what": "Claims an organization unrelated to the email domain",
                    "examples": ["Acme Payroll sent from claim-bonus.example"],
                },
                false={
                    "what": "The identity and domain agree or make no conflicting claim",
                    "examples": ["Acme Payroll sent from acme.example"],
                },
            ),
        ),
        "unexpected_reward": Noul(
            instructions={
                "question": "Does the message announce an unexpected reward?",
                "inspect": "`ticket.message`",
                "focus": "Look for an unsolicited prize, payment, or reward claim.",
            },
            criteria=NoulCriteria(
                true={
                    "what": "Announces an unrequested prize, payment, or reward",
                    "examples": ["You were selected for a $1,000 bonus"],
                },
                false={
                    "what": "Contains no reward claim or discusses an expected payment",
                    "not_for": "A customer asking about a known refund or payroll deposit",
                    "examples": ["When will my approved refund arrive?"],
                },
            ),
        ),
        "refund_requested": Noul(
            instructions={
                "question": "Does the customer explicitly request a refund or credit?",
                "inspect": "`ticket.message`",
                "focus": "Require a requested remedy, not a billing complaint alone.",
            },
            criteria=NoulCriteria(
                true={
                    "what": "Directly asks for money back or an account credit",
                    "examples": ["Please refund the duplicate charge"],
                },
                false={
                    "what": "Does not ask for a refund or credit",
                    "not_for": "A complaint or billing question without a requested remedy",
                    "examples": ["Why was I charged twice?"],
                },
            ),
        ),
        "mentions_open_order": Noul(
            instructions={
                "question": "Does the message refer to a supplied open order?",
                "compare": [
                    "`ticket.message`",
                    "`customer.open_orders`",
                ],
                "focus": "Match an order id or other identifying details.",
            },
            criteria=NoulCriteria(
                true={
                    "what": "Refers to an open order by id or identifying details",
                    "examples": ["Where is order A-104?"],
                },
                false={
                    "what": "Does not identify any supplied open order",
                    "not_for": "A generic order question with no matching details",
                    "examples": ["How long does shipping usually take?"],
                },
            ),
        ),
        "frustration": Score(
            instructions={
                "question": "How frustrated does the customer appear?",
                "inspect": "`ticket.message`",
                "focus": "Judge expressed frustration, not issue severity.",
            },
            criteria=[
                {
                    "what": "Calm and matter-of-fact",
                    "signals": ["Neutral wording", "No complaint about the experience"],
                },
                {
                    "what": "Frustrated but civil",
                    "signals": ["Expresses annoyance", "Remains constructive"],
                },
                {
                    "what": "Very angry or threatening to leave",
                    "signals": ["Hostile language", "Threatens cancellation or churn"],
                },
            ],
        ),
    }

    with TypeSafeClient() as client:
        response = client.system_one(
            state=state,
            questions=questions,
        )

    # Compose independent spam signals with weights controlled by code.
    answers = response.answers
    spam_risk = (
        0.45 * answers["requests_credentials"].noul
        + 0.30 * answers["sender_identity_mismatch"].noul
        + 0.25 * answers["unexpected_reward"].noul
    )

    # Escalate uncertain judgments instead of guessing.
    spam_is_uncertain = 0.4 < spam_risk < 0.6
    if spam_is_uncertain or answers["topic"].confidence < 0.75:
        return route_to_human_review(ticket)
    if spam_risk >= 0.6:
        return quarantine_as_spam(ticket)

    # Let code decide which speculative answers matter on this path.
    if answers["topic"].choice == "billing":
        return route_to_billing(
            ticket,
            refund_requested=answers["refund_requested"].noul >= 0.7,
        )
    if answers["topic"].choice == "orders":
        return route_to_orders(
            ticket,
            mentions_open_order=answers["mentions_open_order"].noul >= 0.7,
        )

    priority = (
        "high"
        if answers["frustration"].confidence >= 0.7
        and answers["frustration"].score >= 1.5
        else "normal"
    )
    return route_to_account_support(ticket, priority=priority)
```

> 中文對照（依程式流程）：
>
> **1. 確定性的狀態不呼叫模型**（「Handle deterministic states without calling a model.」）
> - 工單已關閉 → 直接回傳 `no_action`（不處理）。
> - 用程式碼篩出尚未送達的訂單（`open_orders`）。
>
> **2. 只放入下面問題需要的結構化上下文**（「Include only the structured context needed by the questions below.」）
> - `ticket`：訊息、寄件者、連結。
> - `customer`：方案、未送達的訂單。
> - `policy.sensitive_credentials`：敏感憑證清單（密碼、安全碼、API 金鑰）。
>
> **3. 把結構化、原子化的問題一起提出，讓它們平行執行**（「Ask structured, atomic questions together so they run in parallel.」）
> - `topic`（Choice，主題）：「應該由哪個團隊處理 `ticket.message`？」／依客戶的主要請求分類。
>   - `billing`（帳務）：扣款、發票、退款或訂閱；不適用於訂單追蹤或帳號存取。範例：「我被扣款兩次」、「我的退款在哪？」
>   - `orders`（訂單）：訂單狀態、配送、取消或退貨；不適用於扣款或帳號存取。範例：「我的訂單在哪？」、「取消我的出貨」
>   - `account`（帳號）：登入、個人資料、權限或安全性；不適用於扣款或訂單追蹤。範例：「重設我的密碼」、「我無法登入」
> - `requests_credentials`（Noul，索取敏感憑證）：「這則訊息是否索取敏感憑證？」比對 `ticket.message` 與 `policy.sensitive_credentials`，著重在「要求揭露憑證本身」。
>   - 是：要求收件者揭露清單中的憑證。範例：「回覆你的密碼」、「把你的 API 金鑰寄給我們」
>   - 否：沒有要求揭露憑證；不包括正當的重設憑證指示。範例：「使用這個連結重設你的密碼」
> - `sender_identity_mismatch`（Noul，寄件者身分不符）：「宣稱的寄件者身分是否與其網域相衝突？」比對顯示名稱與電子郵件，著重在「組織名稱與郵件網域是否一致」。
>   - 是：宣稱的組織與郵件網域無關。範例：「Acme 薪資部從 claim-bonus.example 寄出」
>   - 否：身分與網域一致，或沒有互相衝突的宣稱。範例：「Acme 薪資部從 acme.example 寄出」
> - `unexpected_reward`（Noul，意外獎勵）：「這則訊息是否宣布了意外的獎勵？」檢查 `ticket.message`，找出主動送上的獎品、款項或獎勵宣稱。
>   - 是：宣布了未曾要求的獎品、款項或獎勵。範例：「你已獲選領取 1,000 美元獎金」
>   - 否：沒有獎勵宣稱，或討論的是預期中的款項；不包括客戶詢問已知的退款或薪資入帳。範例：「我已核准的退款什麼時候會到？」
> - `refund_requested`（Noul，要求退款）：「客戶是否明確要求退款或抵用額度？」必須是要求補救措施，而不只是帳務抱怨。
>   - 是：直接要求退錢或帳戶抵用額度。範例：「請退還重複的扣款」
>   - 否：沒有要求退款或抵用額度；不包括沒有提出補救要求的抱怨或帳務問題。範例：「為什麼我被扣了兩次？」
> - `mentions_open_order`（Noul，提及未送達訂單）：「這則訊息是否提到所提供的某筆未送達訂單？」比對訊息與 `customer.open_orders`，比對訂單編號或其他可識別的細節。
>   - 是：以編號或可識別的細節提到某筆未送達訂單。範例：「訂單 A-104 在哪？」
>   - 否：沒有指出任何所提供的未送達訂單；不包括沒有相符細節的一般訂單問題。範例：「運送通常要多久？」
> - `frustration`（Score，沮喪程度）：「客戶看起來有多沮喪？」判斷表達出來的沮喪，而不是問題的嚴重程度。
>   - 等級 0：冷靜、就事論事（訊號：中性措辭、沒有抱怨體驗）
>   - 等級 1：沮喪但有禮貌（訊號：表達不滿、仍具建設性）
>   - 等級 2：非常憤怒或揚言離開（訊號：充滿敵意的措辭、揚言取消或流失）
>
> **4. 用程式碼掌控的權重，組合彼此獨立的垃圾訊息訊號**（「Compose independent spam signals with weights controlled by code.」）
> - 垃圾訊息風險 `spam_risk` = 0.45 × 索取憑證 + 0.30 × 寄件者身分不符 + 0.25 × 意外獎勵。
>
> **5. 升級處理不確定的判斷，而不是用猜的**（「Escalate uncertain judgments instead of guessing.」）
> - 垃圾風險介於 0.4 到 0.6（不確定），或主題信心度 < 0.75 → 送交人工審核。
> - 垃圾風險 ≥ 0.6 → 當成垃圾訊息隔離（`quarantine_as_spam`）。
>
> **6. 由程式碼決定在這條路徑上哪些推測性答案才重要**（「Let code decide which speculative answers matter on this path.」）
> - 主題為 `billing`（帳務）→ 轉給帳務，並帶上「是否要求退款」（`noul` ≥ 0.7）。
> - 主題為 `orders`（訂單）→ 轉給訂單團隊，並帶上「是否提及未送達訂單」（`noul` ≥ 0.7）。
> - 其他（`account`，帳號）→ 轉給帳號支援；只有在沮喪程度的信心度 ≥ 0.7 **且**分數 ≥ 1.5 時，優先順序才設為 `high`（高），否則為 `normal`（一般）。
