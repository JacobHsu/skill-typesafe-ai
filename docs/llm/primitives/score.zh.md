> 本文件是 [`score.md`](score.md)（來源：<https://docs.typesafe.ai/primitives/score.md>，下載於 2026-09-23）的繁體中文翻譯，供人類閱讀參考。
> 若內容有出入，以線上英文版本為準。程式碼保持原文，範例中的英文字串另附中文對照。
> 原文包含兩個網站元件的 JavaScript 程式碼：互動示範 `ScoreExplorer`，以及產生 Playground 連結的 `TypesafeExample`。本譯文省略程式碼，前者改以表格說明，後者改寫成 JSON 程式碼區塊。

> ## 文件索引
> 完整的文件索引請見：https://docs.typesafe.ai/llms.txt（中文版：[llms.zh.md](../../llms.zh.md)）
> 在深入探索之前，可先用這份索引了解所有可用的頁面。

# Score

> Score 是一種 System One 問題類型，依照有序且具描述性的等級替內容評分。答案包含分數、每個等級的機率，以及信心度。

當答案是一個「可以分段描述的光譜上的位置」時，就使用 Score。例如：錯誤有多嚴重、客戶有多滿意，或應徵者有多少 Python 經驗。如果答案是一組彼此沒有順序的固定選項之一，請使用 [Choice](choice.zh.md)；如果是「是或否」，請使用 [Noul](noul.zh.md)。[選擇問題類型](../primitives.zh.md#選擇問題類型)一節比較了這三者。

Score 的答案是在你所定義等級上的一個位置，放在 `score` 中，可以落在兩個等級之間。模型也會在 `probabilities` 中回傳每個等級的機率，並為這個答案回傳一個 `confidence`（信心度）值。

**互動示範（ScoreExplorer）**

原網頁在此處有一個互動元件，可切換五個範例，查看每個範例的狀態、等級、機率長條圖、`score` 與 `confidence`：

| 範例 | 問題 | 狀態 | 等級（0 → 最高） | `score` | `confidence` | 機率分布 |
| --- | --- | --- | --- | --- | --- | --- |
| 錯誤嚴重程度（Bug severity） | 所回報的問題有多嚴重？ | Safari 中匯出按鈕讓設定頁當掉；Chrome 正常，但有些客戶只用 Safari。 | 0 外觀問題 / 1 有替代方案 / 2 阻斷性 | 1.43 | 0.35 | 0 / 0.57 / 0.43 |
| 服裝正式程度（Outfit formality） | 根據描述，這套服裝有多正式？ | 海軍藍西裝外套搭素白 T 恤、深色牛仔褲與乾淨的皮革樂福鞋，沒打領帶。 | 0 運動服 / 1 休閒 / 2 商務休閒 / 3 正式 / 4 黑領結 | 1.86 | 0.89 | 0 / 0.14 / 0.86 / 0 / 0 |
| 應徵者契合度（Candidate fit） | 這位應徵者的經驗與職缺有多相關？ | 職缺：打造 Python API 與 PostgreSQL 服務的資深後端工程師。應徵者：三年 Django REST API + PostgreSQL 經驗，之前做過兩年前端 JavaScript；負責過小型服務，但沒帶過後端團隊。 | 0 完全無關 / 1 相鄰領域 / 2 有些直接經驗 / 3 深厚的直接經驗 | 2.52 | 0.52 | 0 / 0 / 0.48 / 0.52 |
| 客戶沮喪程度（Customer frustration） | 客戶有多沮喪？ | 見下方「[將複雜判斷拆成多個 Score 問題](#將複雜判斷拆成多個-score-問題)」的工單。 | 0 冷靜 / 1 沮喪但有禮 / 2 非常憤怒 | 1.26 | 0.61 | 0 / 0.74 / 0.26 |
| 回報詳細程度（Report detail） | 這份回報提供給工程師多少可用資訊？ | 同上。 | 0 沒有細節 / 1 只有功能名稱 / 2 有步驟或環境其一 / 3 步驟與環境都有 | 3.0 | 1.0 | 全部集中在等級 3 |

> （譯註：互動示範中「客戶沮喪程度」的數值為 1.26／0.61，與下方完整請求範例的 1.28／0.58 略有不同，原文如此。）

每個步驟前面的數字代表位置，說明請見下方的[等級](#等級)一節。

## 請求結構

送到 [TypeSafe API](https://docs.typesafe.ai/api.md) 的 POST 請求內容，和其他問題類型一樣有三個頂層欄位：`state`（要評估的內容）、`model`，以及 `questions`。每個 Score 問題有以下欄位：

* `type`：固定為 `"score"`。
* `instructions`：模型要回答的問題，也就是它要評的是什麼。
* `criteria`：一個有序的等級描述陣列，從量表的低端排到高端。至少應有兩個等級；API 最多接受 10 個。

以下是一個請求，狀態是一份錯誤回報，問題是這個錯誤有多嚴重：

**request**

```json
{
  "state": "The export button crashes the settings page in Safari. It works in Chrome, but a few of our customers only use Safari.",
  "model": "jev-latest",
  "questions": {
    "bug_severity": {
      "type": "score",
      "instructions": "How severe is the reported issue?",
      "criteria": [
        "Cosmetic; no impact to functionality",
        "Broken or degraded feature, but workaround exists",
        "Blocking issue; no workaround exists"
      ]
    }
  }
}
```

> 中文對照：
> - 狀態：「匯出按鈕會讓 Safari 中的設定頁當掉。在 Chrome 中正常，但我們有些客戶只用 Safari。」
> - `bug_severity`（錯誤嚴重程度）：「所回報的問題有多嚴重？」
>   - 等級 0：外觀問題；不影響功能
>   - 等級 1：功能損壞或效能下降，但有替代方案
>   - 等級 2：阻斷性問題；沒有替代方案
>
> 原網頁在此範例下方有「Try it in the Playground →」連結，可在 [Playground](https://console.typesafe.ai/playground) 中直接試用。

問題 ID 由你自己決定，這裡是 `bug_severity`。這個 ID 不會傳送給模型；答案會在同一個 ID 底下回傳。

### 等級

`criteria` 中的每一個項目就是一個**等級（level）**：可能答案光譜上的一個點，用文字來描述。等級的編號就是它在 `criteria` 陣列中的位置，從 0 開始，所以上面三個項目分別是等級 0、1、2。陣列的順序就是編號。

模型只會拿到這些描述，其他什麼都沒有；而且每個等級都是**各自獨立**地對照狀態來判斷的。

回應中的 `score` 是在等級光譜上的一個位置。以三個等級的量表來說，它的範圍是 0 到 2，而且可以落在兩個等級之間。

我們的[用戶端 SDK](https://docs.typesafe.ai/sdk.md) 提供具型別的問題。在 Python 中，同一個問題寫成 `Score`：

```python
from typesafe_sdk import Score, TypeSafeClient

with TypeSafeClient() as client:
    response = client.system_one(
        state="The export button crashes the settings page in Safari. It works in Chrome, but a few of our customers only use Safari.",
        questions={
            "bug_severity": Score(
                instructions="How severe is the reported issue?",
                criteria=[
                    "Cosmetic; no impact to functionality",
                    "Broken or degraded feature, but workaround exists",
                    "Blocking issue; no workaround exists",
                ],
            ),
        },
    )

    print(response.answers["bug_severity"].score)
```

使用 `system_one` 方法或 `https://api.typesafe.ai/v1/systemone` 端點來呼叫 System One 模型。`model` 欄位決定由哪個模型處理請求。[如何使用 TypeSafe 建置](../concepts/how-to-build-with-system-one.zh.md)說明了應該在程式碼的哪個位置呼叫它。

你可以使用我們的[用戶端 SDK](https://docs.typesafe.ai/sdk.md)，或直接呼叫 [TypeSafe API](https://docs.typesafe.ai/api.md)。如果是由程式開發 Agent 替你撰寫整合程式，請先安裝 [TypeSafe Agent 技能](https://docs.typesafe.ai/agent-skill.md#installation)，讓它知道請求與回應的格式。

> **注意：** `instructions` 與 `criteria` 中的每個等級都可以是字串、物件或陣列。先從字串開始。當某個等級需要「描述 + 幾個範例情境」時，就使用物件。請見下方的[結構化的等級描述](#結構化的等級描述)，以及 [API 參考](https://docs.typesafe.ai/api.md#param-instructions-2)。

## 回應結構

回應中的 `answers` 對每個問題各有一個項目，放在請求時使用的 ID 底下。以下是上述範例請求的回應：

```json
{
  "model": "jev-1.13.0",
  "answers": {
    "bug_severity": {
      "type": "score",
      "score": 1.43,
      "confidence": 0.35,
      "legend": {
        "0": "Cosmetic; no impact to functionality",
        "1": "Broken or degraded feature, but workaround exists",
        "2": "Blocking issue; no workaround exists"
      },
      "probabilities": {
        "0": 0.0,
        "1": 0.57,
        "2": 0.43
      }
    }
  },
  "usage": {
    "input_tokens": 332,
    "output_tokens": 18
  }
}
```

每個 Score 答案包含五個值：

* `type`：TypeSafe 問題的類型。
* `probabilities`：每個等級的機率，以字串形式的等級編號作為鍵。所有值加總為 1。
* `score`：在等級數線上的位置，範圍從 0 到最高等級的編號（這裡是 2）。它是「每個等級編號 × 其機率」的總和：0 × 0.0 + 1 × 0.57 + 2 × 0.43 = 1.43。
* `legend`：把每個等級編號對應回它的描述。
* [`confidence`](../confidence.zh.md)：一個 0 到 1 的數字，依據 `probabilities` 的分布方式計算而來。機率集中在單一等級上代表高信心度；機率分散在多個等級上代表低信心度。

1.43 分代表模型在等級 1 和等級 2 之間拿不定主意，稍微偏向等級 1。這和回報內容相符：匯出功能壞了，對多數客戶來說改用 Chrome 是個替代方案，但對只用 Safari 的客戶則不是。模型給「有替代方案」0.57、給「沒有替代方案」0.43，而因為意見分歧，信心度是 0.35。

使用 Python SDK 時，`ScoreAnswer` 以具型別的欄位提供 `score`、`confidence`、`probabilities` 與 `legend`。SDK 中 `probabilities` 與 `legend` 的鍵是**整數**等級，而不是字串。

## 解讀 Score

我們來看看分數如何隨不同的輸入而改變。例如，沿用上面請求中的問題與等級：

```
"How severe is the reported issue?"
  → 0: Cosmetic; no impact to functionality
  → 1: Broken or degraded feature, but workaround exists
  → 2: Blocking issue; no workaround exists
```

> 中文對照：「所回報的問題有多嚴重？」→ 0：外觀問題，不影響功能／1：功能損壞或效能下降，但有替代方案／2：阻斷性問題，沒有替代方案

我們可以看到不同的錯誤回報如何改變分數：

| 狀態 | `score` | `confidence` | 等級 0 機率 | 等級 1 機率 | 等級 2 機率 |
| --- | --- | --- | --- | --- | --- |
| The export button is misaligned by a few pixels on the settings page.<br/>（設定頁上的匯出按鈕偏移了幾個像素。） | 0.0 | 1.0 | 1.0 | 0.0 | 0.0 |
| The PDF export button does nothing when clicked. I can still export to CSV and convert it myself, but that takes ages.<br/>（按 PDF 匯出按鈕沒有任何反應。我還是可以匯出成 CSV 再自己轉檔，但那要花很久。） | 1.0 | 1.0 | 0.0 | 1.0 | 0.0 |
| Export to PDF fails with a spinner that never finishes. Some of our team say CSV export still works for them, others say it fails too.<br/>（匯出 PDF 失敗，載入圖示一直轉不停。我們團隊有些人說 CSV 匯出還能用，有些人說也失敗了。） | 1.11 | 0.84 | 0.0 | 0.89 | 0.11 |
| The export button crashes the settings page in Safari. It works in Chrome, but a few of our customers only use Safari.<br/>（匯出按鈕會讓 Safari 中的設定頁當掉。在 Chrome 中正常，但我們有些客戶只用 Safari。） | 1.43 | 0.35 | 0.0 | 0.57 | 0.43 |
| Nobody on our team can log in since this morning. We get a 500 error on every attempt.<br/>（從今天早上開始，我們團隊沒有人能登入。每次嘗試都出現 500 錯誤。） | 2.0 | 1.0 | 0.0 | 0.0 | 1.0 |

在這些範例中，信心度 1.0 代表回傳的分布把所有機率都放在同一個等級上。這描述的是模型的答案，**不保證**答案正確。

分數是等級編號的機率加權平均。在第三和第四個範例中，機率分散在等級 1 和 2 之間；等級 2 的權重越大，分數就越高。它**並不是**在衡量「沒有替代方案的客戶佔多少比例」。

不同的分布可能產生相同的分數。1.0 分可能代表所有機率都在等級 1，也可能代表等級 0 和等級 2 各佔一半。請同時參考 `probabilities` 與 `confidence` 來區分這些情況。

帶小數的分數代表一個位置。你可以用它依嚴重程度替回報排序，或在程式碼需要單一結果時，將它四捨五入到最接近的等級。我們的[實體對齊操作手冊](https://docs.typesafe.ai/cookbooks/entity_alignment.md)展示了一個四捨五入到最近等級來做決策的例子。

Score 的低信心度通常代表以下三種情況之一：對這份狀態而言各等級有所重疊、問題同時在衡量不只一件事，或狀態提供的資訊不足以定位。我們的[信心度](../confidence.zh.md)文件說明了如何在程式碼中運用它。

## 撰寫好的等級

**描述情境，而不是程度。**「功能損壞或效能下降，但有替代方案」給了模型可以對照狀態的具體內容；「中度嚴重」則沒有。具體的描述能幫助模型區分各個等級。請用已知答案的範例來檢查結果；光是信心度變高，並不能證明某個描述比較好。

**每個等級都是分開評估的。** 模型看不到等級的編號，也看不到相鄰的等級，所以「比前一個等級更糟」對它來說毫無意義，在描述或指示中寫數字也沒有幫助。以下是在等級只有數字時，對上表中「按鈕偏移」那份回報的結果：

```
instructions: "Rate severity from 0 to 2, where 2 is worst"
criteria: ["0", "1", "2"]
→ score 0.55, confidence 0.33, probabilities 0: 0.45, 1: 0.55, 2: 0.0
```

> 中文對照：`instructions`＝「以 0 到 2 評定嚴重程度，2 為最嚴重」；`criteria` 只有 `"0"`、`"1"`、`"2"` 三個數字。

同一份回報在使用三個描述性等級時，得到 0.0 分、信心度 1.0。只用數字時，模型沒有可以對照的內容，於是把機率分散在 0 和 1 之間。

**你能清楚區分描述幾個等級，就用幾個等級，最多 10 個。** 三個就很好。不要加入你無法清楚區分描述的等級。

**每個 Score 問題只衡量一個維度。** 如果某個描述寫著「準時、聰明又有經驗」，這個問題就是在衡量三件事；一份在某一項很高、另一項很低的輸入，就無法被定位。信心度會下降，分數的意義也會變弱。請把它拆成「每件事一個 Score 問題」，再在程式碼中組合，如下一節所示。

**如果量表頂端有一種罕見的極端情況，而你需要以不同方式處理它，就給它一個獨立的等級。** 一個以「非常憤怒」為最高等級的情緒量表，可以再加上「辱罵或威脅」。沒有這個等級時，兩種訊息都可能拿到接近頂端的分數，光看分數可能無法區分它們。

**如果完全沒有中間地帶，答案只是少數幾個離散類別之一**，請改用 [Choice](choice.zh.md)，或把問題拆成多個 [Noul](noul.zh.md) 問題。用你自己的資料測試等級非常重要。同一個量表的兩種寫法，在你的資料上可能表現不同。

## 將複雜判斷拆成多個 Score 問題

複雜的判斷，也就是取決於多件事情的判斷，最好拆成「每件事一個 Score 問題」。之後你可以在程式碼中組合 TypeSafe 回傳的各個 Score 來做出判斷。有些 Score 問題可能比其他的更重要，所以要依相對重要性給每個 Score 問題一個權重。權重由你決定。當組合後的結果與你的團隊會做出的決定不一致時，就在程式碼中調整權重再執行一次。請把這些 Score 問題放在同一次請求中送出，它們會平行評估。增加問題幾乎不會影響回應時間，只會多花一些問題本身的 token；請見[一起提出多個問題](../primitives.zh.md#一起提出多個問題)。

下面的請求是上表中「載入圖示轉不停」的那張工單，再加上一些上下文。它提出三個 Score 問題：錯誤有多嚴重、客戶有多沮喪，以及這份回報提供給工程師多少可用資訊。

**request**

```json
{
  "state": "Export to PDF fails with a spinner that never finishes. Some of our team say CSV export still works for them, others say it fails too. This is the third time I'm writing in and honestly I'm done. Steps: open any report, click Export, choose PDF. Chrome 128 on macOS.",
  "model": "jev-latest",
  "questions": {
    "severity": {
      "type": "score",
      "instructions": "How severe is the reported issue?",
      "criteria": [
        "Cosmetic; no impact to functionality",
        "Broken or degraded feature, but workaround exists",
        "Blocking issue; no workaround exists"
      ]
    },
    "frustration": {
      "type": "score",
      "instructions": "How frustrated is the customer?",
      "criteria": [
        "Calm, just stating facts",
        "Frustrated but civil",
        "Very angry, strong language or threatening to leave"
      ]
    },
    "report_quality": {
      "type": "score",
      "instructions": "How much does the report give an engineer to work with?",
      "criteria": [
        "No detail; just says something is broken",
        "Names the feature but no steps or environment",
        "Steps to reproduce or environment, but not both",
        "Steps to reproduce and environment"
      ]
    }
  }
}
```

> 中文對照：
> - 狀態：「匯出 PDF 失敗，載入圖示一直轉不停。我們團隊有些人說 CSV 匯出還能用，有些人說也失敗了。這已經是我第三次來信了，老實說我受夠了。步驟：開啟任一報表，點選匯出，選擇 PDF。macOS 上的 Chrome 128。」
> - `severity`（嚴重程度）：「所回報的問題有多嚴重？」
>   - 等級 0：外觀問題；不影響功能
>   - 等級 1：功能損壞或效能下降，但有替代方案
>   - 等級 2：阻斷性問題；沒有替代方案
> - `frustration`（沮喪程度）：「客戶有多沮喪？」
>   - 等級 0：冷靜，只是陳述事實
>   - 等級 1：沮喪但仍有禮貌
>   - 等級 2：非常憤怒，用詞強烈或揚言離開
> - `report_quality`（回報品質）：「這份回報提供給工程師多少可用資訊？」
>   - 等級 0：沒有細節；只說某個東西壞了
>   - 等級 1：有指出是哪個功能，但沒有步驟或環境資訊
>   - 等級 2：有重現步驟或環境資訊，但不是兩者都有
>   - 等級 3：重現步驟與環境資訊都有
>
> 原網頁在此範例下方有「Try it in the Playground →」連結，可在 [Playground](https://console.typesafe.ai/playground) 中直接試用。

TypeSafe 的回應：

```json
{
  "model": "jev-1.13.0",
  "answers": {
    "severity": {
      "type": "score",
      "score": 1.24,
      "confidence": 0.64,
      "legend": {
        "0": "Cosmetic; no impact to functionality",
        "1": "Broken or degraded feature, but workaround exists",
        "2": "Blocking issue; no workaround exists"
      },
      "probabilities": {
        "0": 0.0,
        "1": 0.76,
        "2": 0.24
      }
    },
    "frustration": {
      "type": "score",
      "score": 1.28,
      "confidence": 0.58,
      "legend": {
        "0": "Calm, just stating facts",
        "1": "Frustrated but civil",
        "2": "Very angry, strong language or threatening to leave"
      },
      "probabilities": {
        "0": 0.0,
        "1": 0.72,
        "2": 0.28
      }
    },
    "report_quality": {
      "type": "score",
      "score": 3.0,
      "confidence": 1.0,
      "legend": {
        "0": "No detail; just says something is broken",
        "1": "Names the feature but no steps or environment",
        "2": "Steps to reproduce or environment, but not both",
        "3": "Steps to reproduce and environment"
      },
      "probabilities": {
        "0": 0.0,
        "1": 0.0,
        "2": 0.0,
        "3": 1.0
      }
    }
  },
  "usage": {
    "input_tokens": 468,
    "output_tokens": 43
  }
}
```

每個問題都各自對照工單作答，並得到一個分數：

* `severity`（嚴重程度）為 1.24，信心度 0.64。判讀和開頭的範例相同：匯出功能壞了，而有些人有替代方案。
* `frustration`（沮喪程度）為 1.28，信心度 0.58。用詞還算有禮貌，但「第三次」和「我受夠了」把部分分數推向最高等級，所以模型在「沮喪但有禮貌」和「非常憤怒」之間分成 0.72 和 0.28。對這張工單而言，這兩個等級有所重疊，這就是信心度只有中等的原因。
* `report_quality`（回報品質）為 3.0，信心度 1.0。重現步驟和瀏覽器版本都有寫明。

這三個量表的長度不同，所以組合之前要先把每個分數正規化。四個等級的量表回傳 0 到 3，三個等級的量表回傳 0 到 2，因此一個量表的最高分會比另一個的最高分大。把每個分數除以它的最高等級編號，也就是 `len(criteria) - 1`，就能讓每個分數都落在 0 到 1。這樣權重才代表它字面上的意思：嚴重程度 0.6、沮喪程度 0.3，就表示嚴重程度的份量是沮喪程度的兩倍。

下面這段使用 TypeSafe Python SDK 的程式碼會提出這三個問題、將每個分數正規化，再用一個範例優先順序公式把它們組合起來：

```python
from typesafe_sdk import Score, TypeSafeClient

TRIAGE_QUESTIONS = {
    "severity": Score(
        instructions="How severe is the reported issue?",
        criteria=[
            "Cosmetic; no impact to functionality",
            "Broken or degraded feature, but workaround exists",
            "Blocking issue; no workaround exists",
        ],
    ),
    "frustration": Score(
        instructions="How frustrated is the customer?",
        criteria=[
            "Calm, just stating facts",
            "Frustrated but civil",
            "Very angry, strong language or threatening to leave",
        ],
    ),
    "report_quality": Score(
        instructions="How much does the report give an engineer to work with?",
        criteria=[
            "No detail; just says something is broken",
            "Names the feature but no steps or environment",
            "Steps to reproduce or environment, but not both",
            "Steps to reproduce and environment",
        ],
    ),
}


def normalized(answers, question_id: str) -> float:
    """Put a score on 0 to 1 by dividing by its top level number."""
    top_level = len(TRIAGE_QUESTIONS[question_id].criteria) - 1
    return answers[question_id].score / top_level


def priority(ticket: str) -> float:
    with TypeSafeClient() as client:
        response = client.system_one(
            state=ticket,
            questions=TRIAGE_QUESTIONS,
        )
    answers = response.answers

    severity = normalized(answers, "severity")
    frustration = normalized(answers, "frustration")
    report_quality = normalized(answers, "report_quality")

    # A detailed report helps an engineer investigate, so it raises priority a little.
    return 0.6 * severity + 0.3 * frustration + 0.1 * report_quality
```

> 中文對照（程式註解）：
> - `normalized` 的說明：「把分數除以其最高等級編號，使其落在 0 到 1。」
> - `# A detailed report helps…`：「詳細的回報有助於工程師調查，所以會稍微提高優先順序。」

以上面的範例回應來說，正規化後的分數為：嚴重程度 0.62、沮喪程度 0.64、回報品質 1.0。優先順序為 `0.6 × 0.62 + 0.3 × 0.64 + 0.1 × 1.0 = 0.664`，四捨五入為 `0.66`。

權重存在於你的程式碼中，所以你能清楚看到這個數字是怎麼算出來的，並在排序結果不符合你團隊的做法時修改它。如果之後需要更多 Score 問題，把它們加進 `TRIAGE_QUESTIONS` 即可，請求次數仍然只有一次。這種把複雜判斷拆成多個獨立 Score、再在程式碼中用權重組合的技巧，稱為[複合評分（Composite scoring）](../patterns/composite-scoring.zh.md)模式。

## 結構化的等級描述

先從每個等級一段簡單的文字描述開始。當你認為某些輸入很明確，模型卻一直在兩個相鄰等級之間給分時，就把每個等級從字串改成物件：一個欄位寫這個等級涵蓋什麼，另一個欄位列出幾個範例情境。每個等級都使用相同的欄位名稱，讓模型能拿同類的東西互相比較。

下面的請求就是先前用過的「載入圖示轉不停」工單，但在每個等級都加上了範例：

**request**

```json
{
  "state": "Export to PDF fails with a spinner that never finishes. Some of our team say CSV export still works for them, others say it fails too.",
  "model": "jev-latest",
  "questions": {
    "bug_severity": {
      "type": "score",
      "instructions": "How severe is the reported issue?",
      "criteria": [
        {
          "what": "Cosmetic; no impact to functionality",
          "examples": ["typo in a label", "misaligned icon"]
        },
        {
          "what": "Broken or degraded feature, but workaround exists",
          "examples": ["export fails in one browser but works in another"]
        },
        {
          "what": "Blocking issue; no workaround exists",
          "examples": ["cannot log in", "data loss"]
        }
      ]
    }
  }
}
```

> 中文對照：
> - 等級 0：`what`＝外觀問題；不影響功能。`examples`＝標籤有錯字、圖示沒對齊
> - 等級 1：`what`＝功能損壞或效能下降，但有替代方案。`examples`＝匯出在某個瀏覽器失敗，但在另一個瀏覽器可以
> - 等級 2：`what`＝阻斷性問題；沒有替代方案。`examples`＝無法登入、資料遺失
>
> 原網頁在此範例下方有「Try it in the Playground →」連結，可在 [Playground](https://console.typesafe.ai/playground) 中直接試用。

回應：

```json
{
  "model": "jev-1.13.0",
  "answers": {
    "bug_severity": {
      "type": "score",
      "score": 1.09,
      "confidence": 0.87,
      "legend": {
        "0": {
          "what": "Cosmetic; no impact to functionality",
          "examples": [
            "typo in a label",
            "misaligned icon"
          ]
        },
        "1": {
          "what": "Broken or degraded feature, but workaround exists",
          "examples": [
            "export fails in one browser but works in another"
          ]
        },
        "2": {
          "what": "Blocking issue; no workaround exists",
          "examples": [
            "cannot log in",
            "data loss"
          ]
        }
      },
      "probabilities": {
        "0": 0.0,
        "1": 0.91,
        "2": 0.09
      }
    }
  },
  "usage": {
    "input_tokens": 379,
    "output_tokens": 18
  }
}
```

使用純字串時，這張工單得到 1.11 分、信心度 0.84；加上範例後，得到 1.09 分、信心度 0.87。變化很小，因為純字串原本就已經把它定位得不錯了。當純字串讓模型拿不定主意時，效果會更明顯，如下表所示。

範例會引導模型，而且只有在它們長得像你真實的輸入時才有幫助。下表是開頭那份 Safari 回報，搭配三組不同的等級物件：

| 等級描述 | `score` | `confidence` |
| --- | --- | --- |
| 純字串：沒有附範例的物件 | 1.43 | 0.35 |
| 加上範例陣列，放入有用的範例：「匯出在某個瀏覽器失敗，但在另一個瀏覽器可以」 | 1.03 | 0.96 |
| 加上範例陣列，放入與瀏覽器無關的範例：「搜尋失敗，但瀏覽分類仍然正常」 | 1.43 | 0.35 |

在這個比較中，相符的範例幾乎把所有機率都集中到同一個等級上；不相關的範例則和純字串的結果一模一樣。信心度變高並不能證明哪個答案才是正確的。請挑選已知預期等級的範例，並在保留修改後的描述之前，先用另一批輸入加以測試。
