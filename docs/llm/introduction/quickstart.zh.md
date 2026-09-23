> 本文件是 [`quickstart.md`](quickstart.md)（來源：<https://docs.typesafe.ai/introduction/quickstart.md>，下載於 2026-09-23）的繁體中文翻譯，供人類閱讀參考。
> 若內容有出入，以線上英文版本為準。程式碼保持原文，範例中的英文字串另附中文對照。

> ## 文件索引
> 完整的文件索引請見：https://docs.typesafe.ai/llms.txt（中文版：[llms.zh.md](../../llms.zh.md)）
> 在深入探索之前，可先用這份索引了解所有可用的頁面。

# 快速開始

> 想直接上手？這裡有立即開始所需的一切。

## 試試看：Playground

1. **開啟 [Playground](https://console.typesafe.ai/playground)** 並登入。
2. **貼上任意文字**作為狀態（state）。

**範例狀態**

```plaintext
Hi, I've been trying to connect my Stripe account for 3 days and the integration keeps failing. I'm losing sales. Please help ASAP.
```

> 中文對照：「嗨，我已經試著連接我的 Stripe 帳號三天了，整合一直失敗。我正在流失業績。請盡快幫忙。」

3. **新增一個問題。** 試試 Noul 問題：`"Does this message express urgency?"`（這則訊息是否表達了緊急性？）

```json
{
  "urgency": {
    "type": "noul",
    "instructions": "Does this message express urgency?"
  }
}
```

4. **新增更多問題。** 在一次呼叫中混合使用 Noul、Choice 與 Score，一次看到所有結果。

## 呼叫它：API

1. 從[控制台](https://console.typesafe.ai/keys)**取得你的 API 金鑰**。
2. 對 API 端點**發送 POST 請求**。
3. 所有細節請**參閱 [API 參考](https://docs.typesafe.ai/api.md)**。

```http
POST https://api.typesafe.ai/v1/systemone
Authorization: Bearer <API_KEY>
Content-Type: application/json
```

### cURL 指令範例

```bash
curl -X POST https://api.typesafe.ai/v1/systemone \
  -H "Authorization: Bearer $TYPESAFE_API_KEY" \
  -H "Content-Type: application/json" \
  -d @- <<'EOF'
  {
    "state": "Hi, I've been trying to connect my Stripe account for 3 days and the integration keeps failing. I'm losing sales. Please help ASAP.",
    "model": "jev-latest",
    "questions": {
      "urgency": {
        "type": "noul",
        "instructions": "Does this message express urgency?"
      }
    }
  }
EOF
```

### 請求內容（Request body）

```json
{
  "state": "Hi, I've been trying to connect my Stripe account for 3 days and the integration keeps failing. I'm losing sales. Please help ASAP.",
  "model": "jev-latest",
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
    "frustration": {
      "type": "score",
      "instructions": "How frustrated the customer appears",
      "criteria": [
        "Calm, just stating facts",
        "Frustrated but civil",
        "Very angry, strong language"
      ]
    },
    "is_urgent": {
      "type": "noul",
      "instructions": "The message conveys urgency or time-sensitivity"
    }
  }
}
```

> 中文對照：
> - `department`（Choice）：「應該由哪個團隊處理」
>   - `billing`：付款或訂閱問題
>   - `technical`：錯誤或整合問題
>   - `sales`：價格或帳號問題
> - `frustration`（Score）：「客戶看起來有多沮喪」
>   - 等級 0：冷靜，只是陳述事實
>   - 等級 1：沮喪但仍有禮貌
>   - 等級 2：非常憤怒，用詞強烈
> - `is_urgent`（Noul）：「這則訊息傳達了緊急性或時間壓力」

### 回應內容（Response body）

```json
{
  "model": "jev-1.13.0",
  "answers": {
    "department": {
      "type": "choice",
      "choice": "technical",
      "confidence": 0.78,
      "probabilities": {
        "technical": 0.85,
        "sales": 0.0,
        "billing": 0.15
      }
    },
    "frustration": {
      "type": "score",
      "score": 1.0,
      "confidence": 1.0,
      "legend": {
        "0": "Calm, just stating facts",
        "1": "Frustrated but civil",
        "2": "Very angry, strong language"
      },
      "probabilities": {
        "0": 0.0,
        "1": 1.0,
        "2": 0.0
      }
    },
    "is_urgent": {
      "type": "noul",
      "noul": 1.0
    }
  },
  "usage": {
    "input_tokens": 392,
    "output_tokens": 65
  }
}
```

所有細節請參閱 [API 參考](https://docs.typesafe.ai/api.md)。

## 寫程式：Python SDK

1. **安裝 SDK**（需要 Python >= 3.10）。

**使用 pip**

```bash
pip install typesafe-sdk
```

**使用 uv**

```bash
uv add typesafe-sdk
```

2. **使用 SDK。** 用戶端會從環境變數讀取 `TYPESAFE_API_KEY`，並預設呼叫 `jev-latest`。

```python
from typesafe_sdk import Choice, Noul, Score, TypeSafeClient

client = TypeSafeClient()

ticket = "Hi, I've been trying to connect my Stripe account for 3 days and the integration keeps failing. I'm losing sales. Please help ASAP."

response = client.system_one(
    state=ticket,
    questions={
        "department": Choice(
            instructions="Which team should handle this",
            criteria={
                "billing": "Payment or subscription issues",
                "technical": "Bugs or integration problems",
                "sales": "Pricing or account questions",
            },
        ),
        "frustration": Score(
            instructions="How frustrated the customer appears",
            criteria=[
                "Calm, just stating facts",
                "Frustrated but civil",
                "Very angry, strong language",
            ],
        ),
        "is_urgent": Noul(
            instructions="The message conveys urgency or time-sensitivity",
        ),
    },
)

print(response.answers["department"].choice)  # "technical"
print(response.answers["frustration"].score)  # 1.0
print(response.answers["is_urgent"].noul)     # 1.0
```

安裝選項與詳細用法請參閱[用戶端 SDK](https://docs.typesafe.ai/sdk.md)。

## 交給 Agent：Agent 技能

1. 使用 Claude Code 外掛或 `npx skills add typesafe-ai/skills --skill typesafe-ai` **[安裝 TypeSafe 技能](https://docs.typesafe.ai/agent-skill.md#installation)**。你也可以[在 GitHub 上閱讀 SKILL.md](https://github.com/typesafe-ai/skills/blob/main/skills/typesafe-ai/SKILL.md)（中文版：[SKILL.zh.md](../../SKILL.zh.md)）。

   **方式一：Claude Code**

   在終端機執行以下兩個指令：

   ```bash
   claude plugin marketplace add typesafe-ai/skills
   claude plugin install typesafe@typesafe-ai
   ```

   **方式二：其他 Agent**

   ```bash
   npx skills add typesafe-ai/skills --skill typesafe-ai
   ```

   請在提示時選擇你的 Agent。預設會以專案為範圍安裝；若要全域安裝，請加上 `-g`。

   **方式三：複製給你的 Agent**

   將以下提示詞貼到你的程式開發 Agent：

   ```text
   Install the TypeSafe skill. If you're in Claude Code, run `claude plugin marketplace add typesafe-ai/skills`, then `claude plugin install typesafe@typesafe-ai`. If you're in another agent, run `npx skills add typesafe-ai/skills --skill typesafe-ai` and select your agent. Use one installation method. You can read the skill directly at https://github.com/typesafe-ai/skills/blob/main/skills/typesafe-ai/SKILL.md (raw: https://raw.githubusercontent.com/typesafe-ai/skills/main/skills/typesafe-ai/SKILL.md). Then use the TypeSafe skill when working on this project.
   ```

   > 中文對照：「安裝 TypeSafe 技能。如果你在 Claude Code 中，先執行 `claude plugin marketplace add typesafe-ai/skills`，再執行 `claude plugin install typesafe@typesafe-ai`。如果你在其他 Agent 中，執行 `npx skills add typesafe-ai/skills --skill typesafe-ai` 並選擇你的 Agent。只使用一種安裝方式。你可以直接在 https://github.com/typesafe-ai/skills/blob/main/skills/typesafe-ai/SKILL.md 閱讀這個技能（原始檔：https://raw.githubusercontent.com/typesafe-ai/skills/main/skills/typesafe-ai/SKILL.md）。之後在這個專案中工作時，請使用 TypeSafe 技能。」

2. 在建置過程中，**告訴你的程式開發 Agent** 使用 TypeSafe 技能！

**程式開發 Agent 提示詞**

```plaintext
Let's build a simple CLI that uses the TypeSafe API to evaluate a set of supplied documents on multiple dimensions. Use the TypeSafe skill to understand how to use the TypeSafe API and how to structure the system. Ask me questions about what kinds of documents I want to evaluate and on what dimensions.
```

> 中文對照：「我們來建置一個簡單的 CLI，使用 TypeSafe API 從多個維度評估一組提供的文件。請使用 TypeSafe 技能來了解如何使用 TypeSafe API，以及如何架構這個系統。請問我想評估哪些類型的文件、以及要從哪些維度評估。」

更多細節請參閱 [Agent 技能](https://docs.typesafe.ai/agent-skill.md)頁面。
