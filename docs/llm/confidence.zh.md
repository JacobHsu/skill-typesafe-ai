> 本文件是 [`confidence.md`](confidence.md)（來源：<https://docs.typesafe.ai/confidence.md>，下載於 2026-09-23）的繁體中文翻譯，供人類閱讀參考。
> 若內容有出入，以線上英文版本為準。程式碼保持原文，範例中的英文字串另附中文對照。
> 原文包含一個互動元件（`ConfidenceExplorer`）的 JavaScript 程式碼；本譯文省略程式碼，改以文字說明其內容。

> ## 文件索引
> 完整的文件索引請見：https://docs.typesafe.ai/llms.txt（中文版：[llms.zh.md](../llms.zh.md)）
> 在深入探索之前，可先用這份索引了解所有可用的頁面。

# 信心度（Confidence）

> TypeSafe 如何回報確定程度、它與機率有何不同，以及如何用它來控制系統行為。

TypeSafe 回傳的所有 Score 與 Choice 答案，都包含一個 `probabilities` 屬性，代表在各選項（Choice）或各等級（Score）上的機率分布。這個分布的**形狀**告訴你模型有多確定：集中在單一結果上，代表答案很有把握；分散開來，則代表不確定。

答案的 `confidence` 屬性把這個形狀壓縮成一個 0 到 1 的數字，讓你不必自己計算就能直接拿它設門檻。（Noul 的答案沒有這個屬性。）

## 信心度是從機率推導而來的

`confidence` 是根據答案本來就會給你的機率分布所計算出的統計量。TypeSafe 會替你算好，並在每個 Choice 與 Score 答案中回傳，所以一般情況下你不需要做任何額外的工作。

**互動示範：三個選項的 Choice 問題——看看機率分布如何改變信心度**

原網頁在此處有一個互動元件：三個滑桿分別控制選項 A、B、C 的機率（調整其中一個時，其他兩個會自動調整，使總和維持 100%），右側即時顯示信心度與目前選中的選項。另有三個預設按鈕：

| 預設 | A / B / C 的機率 | 信心度 |
| --- | --- | --- |
| 明顯勝出（Clear winner） | 90% / 6% / 4% | 0.85 |
| 分散（Spread out） | 40% / 33% / 27% | 0.10 |
| 平均分配（Even split） | 33⅓% / 33⅓% / 33⅓% | 0 |

> **這個示範如何計算信心度：** TypeSafe 依據機率在各選項間的分布方式來計算信心度。機率全部集中在單一選項時為 1.0；分布越平均，信心度越低。這個示範使用 `(3 × 最大機率 − 1) / 2` 來近似三個選項時的信心度。
>
> （譯註：信心度欄的數值為譯者依此公式計算，方便對照。以[快速開始](introduction/quickstart.zh.md)的回應為例，`department` 的最大機率為 0.85，(3 × 0.85 − 1) / 2 ≈ 0.78，正好等於回應中的 `confidence`。）

> **注意——一個可靠的預設值：** 我們提供 `confidence` 作為適用於大多數使用情境的便利指標，但你永遠不會被綁死在我們的定義上。依你所評估的內容而定，其他指標可能更適合你，這正是我們在回應中提供完整 `probabilities` 的原因。不同計算方式的優缺點是一個專門的主題，我們會另外寫成一份操作手冊，而不放在本頁；完成後會在這裡加上連結！

對 [Choice](primitives/choice.zh.md) 而言，分布指的是你所有選項上的 `probabilities`；對 [Score](primitives/score.zh.md) 而言，則是你所有等級上的分布。在這兩種情況下，分布越平坦，信心度越低：Choice 的低信心度通常表示沒有任何選項明顯勝過其他選項；Score 的低信心度則通常表示等級定義模糊、混雜了多個維度，或是狀態中的資訊不足以做出判斷。

## 「我不知道」是有用的訊號

一個智慧系統，無論是人還是機器，如果無法誠實地表達不確定性，這個系統就無法被信任。

信心度提供了一種內建機制，讓模型能夠說出「這一題我不太確定」。這讓你的程式碼可以針對不同的確定程度實作不同的行為，而這正是打造真正可以依賴的系統的基礎。

## 在程式碼中運用信心度的三條路徑

一個實用的起始模式是把信心度分成三個區間，每個區間產生不同的系統行為：

**高信心度：** 自動執行。模型有清楚的判讀，你可以在不需要人工介入的情況下繼續進行。

**中信心度：** 謹慎進行。模型給出了合理的答案，但並不確定。視情境而定，你可以請使用者確認、標記送審，或在行動前蒐集更多資訊。

**低信心度：** 不要行動。轉交人工、請對方補充說明，或改用其他系統處理。模型是在告訴你：它沒有足夠的資訊，或這個問題不適合它。

這些界線要畫在哪裡，取決於事情的利害輕重。

## 門檻隨風險調整

信心度門檻不是單一數字。同一個系統中的不同動作，應依據「做錯的後果」設在不同的門檻上。

```python
response = client.system_one(
    state=user_message,
    questions={
        "action": Choice(
            instructions="What is the user trying to do?",
            criteria={
                "check_balance": "View account balance",
                "approve_transfer": "Approve the pending withdrawal request",
                "support": "Get help with an issue",
            },
        ),
    },
)

action = response.answers["action"]
confidence = action.confidence

if confidence < 0.5:
    # Model is genuinely unsure. Don't guess.
    route_to_human(user_message)

elif action.choice == "check_balance":
    # Low stakes. Showing the wrong screen is recoverable.
    show_balance(account_id)

elif action.choice == "approve_transfer":
    if confidence > 0.9:
        # High stakes, high confidence. Proceed with confirmation.
        confirm_then_execute(account_id)
    else:
        # High stakes, moderate confidence. Verify first.
        ask_user_to_confirm(account_id)
```

> 中文對照：
> - `action`（Choice）：「使用者想要做什麼？」
>   - `check_balance`（查詢餘額）：檢視帳戶餘額
>   - `approve_transfer`（核准轉帳）：核准待處理的提款請求
>   - `support`（客服支援）：針對某個問題尋求協助
> - 程式註解：
>   - 信心度 < 0.5：「模型真的不確定。不要用猜的。」→ 轉交人工（`route_to_human`）
>   - `check_balance`：「風險低。顯示錯的畫面是可以挽回的。」→ 顯示餘額（`show_balance`）
>   - `approve_transfer` 且信心度 > 0.9：「高風險、高信心度。在確認後執行。」→ `confirm_then_execute`
>   - `approve_transfer` 且信心度 ≤ 0.9：「高風險、中等信心度。先驗證。」→ 請使用者確認（`ask_user_to_confirm`）

0.5 的信心度下限，會攔下模型回報為真正不確定的所有情況。在這之上，破壞性操作（例如轉帳）不經確認就執行的門檻，要比唯讀操作（例如查餘額）更高。你的程式碼負責表達你的風險容忍度。

> **注意：** 正確的門檻值取決於你的領域，以及模型在你的使用情境中的表現。從保守的門檻開始，用你自己的資料測試，再依觀察到的結果調整。
