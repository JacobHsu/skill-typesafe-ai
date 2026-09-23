# TypeSafe AI 文件索引（繁體中文版）

> 本文件是 [`llms.txt`](llms.txt)（來源：<https://docs.typesafe.ai/llms.txt>，下載於 2026-09-23）的繁體中文翻譯，供人類閱讀參考。
> 連結指向的原始文件皆為英文；線上索引可能已更新，若內容有出入，以線上版本為準。

> 如何使用 TypeSafe 的 System One API

## 入門與核心概念

- [簡介](llm/introduction.zh.md)（[原文](https://docs.typesafe.ai/introduction.md)）：Jev 是 TypeSafe 的旗艦模型，也是第一個 System One 模型。送出狀態與具型別的問題，即可取得程式碼能直接使用的結構化答案。
- [快速開始](llm/introduction/quickstart.zh.md)（[原文](https://docs.typesafe.ai/introduction/quickstart.md)）：想直接上手？這裡有立即開始所需的一切。
- [搭配程式開發 Agent 使用 Jev](https://docs.typesafe.ai/introduction/coding-agents.md)：使用程式開發 Agent 時，Jev 是什麼（以及不是什麼）。
- [使用案例範例](https://docs.typesafe.ai/concepts/use-case-map.md)：依產業探索 TypeSafe 的使用案例，並將有潛力的構想轉化為軟體工作流程。
- [System One](llm/concepts/system-one.zh.md)（[原文](https://docs.typesafe.ai/concepts/system-one.md)）：System One 模型為軟體做出快速、結構化的決策。Jev 是 TypeSafe 的旗艦模型，也是第一個 System One 模型。
- [狀態（State）](llm/concepts/state.zh.md)（[原文](https://docs.typesafe.ai/concepts/state.md)）：什麼是狀態、如何組織它，以及如何提供 System One 模型所需的上下文。
- [原語（問題）](llm/primitives.zh.md)（[原文](https://docs.typesafe.ai/primitives.md)）：TypeSafe 的三種問題類型（Choice、Score、Noul）、它們回傳的具型別答案、如何在三者之間選擇，以及如何一次提出多個問題。
- [Choice](llm/primitives/choice.zh.md)（[原文](https://docs.typesafe.ai/primitives/choice.md)）：Choice 是一種 System One 問題類型，用於從既定集合中選出一個選項。答案包含選中的選項、每個選項的機率，以及信心度。
- [Score](llm/primitives/score.zh.md)（[原文](https://docs.typesafe.ai/primitives/score.md)）：Score 是一種 System One 問題類型，依照有序且具描述性的等級替內容評分。答案包含分數、每個等級的機率，以及信心度。
- [Noul](llm/primitives/noul.zh.md)（[原文](https://docs.typesafe.ai/primitives/noul.md)）：Noul 問題要求 TypeSafe 模型評估一個是非題，並回傳答案為「是」的機率。
- [進階：結構](https://docs.typesafe.ai/primitives/advanced.md)：指示（instructions）、Choice 選項、Score 等級與 Noul 準則都接受 JSON 結構。
- [信心度（Confidence）](llm/confidence.zh.md)（[原文](https://docs.typesafe.ai/confidence.md)）：TypeSafe 如何回報確定程度、它與機率有何不同，以及如何用它來控制系統行為。
- [如何使用 TypeSafe 建置](llm/concepts/how-to-build-with-system-one.zh.md)（[原文](https://docs.typesafe.ai/concepts/how-to-build-with-system-one.md)）：設計 AI 驅動的軟體時，讓程式碼保有掌控權，只把範圍明確、結構化的決策交給 System One。
- [AI 入門](https://docs.typesafe.ai/introduction/machine-learning-primer.md)：為什麼 TypeSafe 以經過校準的機率來訓練決策模型，而不是針對生成文字進行最佳化。

## 模式

- [模式](https://docs.typesafe.ai/patterns.md)：使用 TypeSafe 建置系統的架構模式。
- [推測式扇出（Speculative fan-out）](llm/patterns/fan-out.zh.md)（[原文](https://docs.typesafe.ai/patterns/fan-out.md)）：在單一呼叫中送出多個問題（包括推測性的問題），再由程式碼決定哪些答案相關。
- [信心度門檻路由](llm/patterns/confidence-routing.zh.md)（[原文](https://docs.typesafe.ai/patterns/confidence-routing.md)）：把信心度當成第二個維度。答案告訴你「是什麼」，信心度告訴你「要不要行動」。
- [複合評分](llm/patterns/composite-scoring.zh.md)（[原文](https://docs.typesafe.ai/patterns/composite-scoring.md)）：將複雜的判斷拆解成原子化的分數，再用程式碼中可控的權重加以組合。
- [意圖路由](https://docs.typesafe.ai/patterns/intent-routing.md)：將傳入的請求分類，並把每一個路由到最適合的處理者：確定性邏輯、專門的 LLM，或是人工。

## 操作手冊（Cookbooks）

- [操作手冊](https://docs.typesafe.ai/cookbooks.md)：端對端的實作範例，展示 TypeSafe 如何解決實際問題，從幾個問題到完整的處理流程都有。
- [自我一致性：Noul](https://docs.typesafe.ai/cookbooks/consistency_noul_cookbook.md)：將不確定的機率轉交人工審核，同時保留底層 Noul 數值的可見性。
- [自我一致性：Choice](https://docs.typesafe.ai/cookbooks/consistency_choice_cookbook.md)：在內容審核決策中加入「不確定」結果，並比較標籤一致率與自動處理的比例。
- [平行提問](https://docs.typesafe.ai/cookbooks/parallel_questions.md)：針對 GDPR 維基百科條目執行 13 個問題的法規簡報，顯示將所有問題批次放進一次 TypeSafe 呼叫，成本降低 12.2 倍、速度快 10.0 倍，且答案不變。
- [重新排序（Re-ranking）](https://docs.typesafe.ai/cookbooks/rerank_typesafe.md)：為 40 個 CLERC 法律查詢建立 30 段落的 BM25 候選清單，再對每組「查詢—候選」配對使用一個 TypeSafe 問題，將 top-1 準確率從 5% 提升到 18%，top-10 準確率從 38% 提升到 62%。
- [逐行搜尋](https://docs.typesafe.ai/cookbooks/semantic_find.md)：為 GitHub 服務條款建置語意搜尋。在一次請求中，用 Choice 問題依白話查詢替 218 個行 ID 評分，並用 Noul 問題檢查文件中是否包含答案。
- [結構還原](https://docs.typesafe.ai/cookbooks/autoformat.md)：以兩次請求將失去格式的純文字重建為 Markdown：一次把被硬換行切斷的行接回去，一次替每個區塊分類（標題、清單、程式碼、提示框）。
- [函式呼叫](https://docs.typesafe.ai/cookbooks/function_calling.md)：將函式名稱與封閉集合的參數對應到具信心度判斷的 TypeSafe 問題，把自然語言的交易請求轉換為一般具型別函式的呼叫。
- [技能建議](https://docs.typesafe.ai/cookbooks/skill_suggestion.md)：從 Nous Research 的 Hermes 目錄中 182 個技能裡，為 Agent 的某一輪對話最多挑選一個技能，使用兩次 TypeSafe 請求來排序並複查排名最前的候選。
- [知識圖譜實體對齊](https://docs.typesafe.ai/cookbooks/entity_alignment.md)：判斷來自兩份啤酒目錄的 450 組候選配對中，哪些描述的是同一個產品；使用一個 Score 問題，搭配三個 Noul 問題指出哪些欄位不一致。
- [RAG 段落分類](https://docs.typesafe.ai/cookbooks/classifying_rag_passages.md)：用一次 TypeSafe 請求替每個檢索到的段落評分，再由程式碼決定哪些段落送進負責回答的模型。
- [複查引用](https://docs.typesafe.ai/cookbooks/citation_check.md)：對照來源文件，揪出錯誤或幻覺產生的引用。一個 Choice 問題判斷引文的上下文是否支持該主張。
- [LLM 防護機制](https://docs.typesafe.ai/cookbooks/llm_guardrails.md)：用一次 TypeSafe 請求篩檢 LLM 應用程式的每一則輸入與輸出訊息，依危害機率與嚴重程度設定門檻，決定放行、審核、封鎖或轉送。
- [SDE 串接](https://docs.typesafe.ai/cookbooks/sde_cascade.md)：使用兩階段的結構化資料擷取（SDE）串接流程（小模型 → 驗證 → 推理模型），以一小部分的成本取得接近大型推理模型的品質。
- [日期擷取](https://docs.typesafe.ai/cookbooks/date_extraction_cookbook.md)：請 TypeSafe 找出文件中提及的日期組成部分，再由程式碼解析並驗證，並依信心度進行審核，藉此擷取絕對與相對日期。
- [預先解析的數值擷取](https://docs.typesafe.ai/cookbooks/pre_parsed_value_extraction_cookbook.md)：用正規表示式找出候選的電子郵件、電話號碼與金額，再由 TypeSafe 選出所要的片段，讓程式碼能將原文數值正規化。
- [階層式分類](https://docs.typesafe.ai/cookbooks/hierarchical_classification.md)：透過對 TypeSafe Choice 機率進行平行束搜尋（beam search），將文件分類到深層的專利、零售商品、生物醫學與原始碼階層中。
- [Autoresearch 特徵探索](https://docs.typesafe.ai/cookbooks/autoresearch_feature_discovery.md)：執行自動研究迴圈，提出 TypeSafe 問題、將自由文字轉換為數值特徵，並利用模型誤差來改進監督式 CatBoost 迴歸模型。
- [利用信心度進行分類](https://docs.typesafe.ai/cookbooks/classification_using_confidence.md)：以每份一個 Choice 問題，將 SEC 年報分類到 75 個產業群組，再依答案本身的信心度，決定回報該群組還是其上一層的較大類別。

## 示範

- [示範](https://docs.typesafe.ai/demos.md)：互動式範例，展示 TypeSafe 能做到的事。
- [智慧家庭助理示範](https://docs.typesafe.ai/demos/smart-home.md)：示範程式碼：一個使用 TypeSafe 評估使用者請求的智慧家庭助理。

## 參考資料

- [模型](https://docs.typesafe.ai/models.md)
- [API 參考](https://docs.typesafe.ai/api.md)：TypeSafe 評估端點的完整 HTTP API 參考文件。
- [Agent 技能](https://docs.typesafe.ai/agent-skill.md)：可直接放入 Claude Code、Codex 與其他 Agent 環境使用的技能。
- [法律](https://docs.typesafe.ai/legal.md)：TypeSafe 的法律文件與政策。
- [Jev 1.13 的不平整之處](https://docs.typesafe.ai/model-jaggedness/jev-1.13.md)：Jev 並不完美。這裡列出我們已知 jev-1.13 的一些不平整之處（jagged edges），其中許多會在後續版本中修正。

## 用戶端 SDK

- [用戶端 SDK](https://docs.typesafe.ai/sdk.md)：安裝 TypeSafe 用戶端 SDK，並在應用程式中使用具型別的問題與答案。

### Python SDK

- [TypeSafe Python SDK](https://docs.typesafe.ai/sdk/python.md)：安裝 TypeSafe Python SDK，並開始使用非同步或同步的 API 呼叫。
- [用法](https://docs.typesafe.ai/sdk/python/usage.md)：使用 TypeSafe Python SDK 的指南與模式。
- [更新紀錄](https://docs.typesafe.ai/sdk/python/changelog.md)：TypeSafe AI API 的 Python 用戶端
- [API 參考](https://docs.typesafe.ai/sdk/python/api.md)：TypeSafe AI API 的 Python 用戶端
- [非同步用戶端](https://docs.typesafe.ai/sdk/python/api/clients/async.md)：使用 AsyncTypeSafeClient 提出問題、列出模型，並設定非同步的 TypeSafe API 請求。
- [同步用戶端](https://docs.typesafe.ai/sdk/python/api/clients/sync.md)：使用 TypeSafeClient 提出問題、列出模型，並設定同步的 TypeSafe API 請求。
- [問題](https://docs.typesafe.ai/sdk/python/api/types/questions.md)：以物件或字典提供狀態，並提出是非題、選擇題與評分題。
- [答案與回應](https://docs.typesafe.ai/sdk/python/api/types/responses.md)：讀取 TypeSafe API 回傳的答案、信心分數、token 用量與可用模型。
- [重試](https://docs.typesafe.ai/sdk/python/api/retries.md)：以 RetryPolicy 設定重試行為：嘗試次數、可重試的狀態碼、退避策略，以及重試相關標頭的處理。
- [通用型別](https://docs.typesafe.ai/sdk/python/api/types/common.md)：TypeSafe API SDK 的通用型別。
- [例外](https://docs.typesafe.ai/sdk/python/api/exceptions.md)：處理 TypeSafe API 錯誤、速率限制、連線失敗與逾時。
- [常數](https://docs.typesafe.ai/sdk/python/api/constants.md)：TypeSafe Python SDK 的預設設定與環境變數名稱。

### JavaScript SDK

- [JavaScript SDK](https://docs.typesafe.ai/sdk/javascript.md)
- [更新紀錄](https://docs.typesafe.ai/sdk/javascript/changelog.md)
- [API 參考](https://docs.typesafe.ai/sdk/javascript/api.md)

#### 類別（Class）

- [APIConnectionError](https://docs.typesafe.ai/sdk/javascript/api/classes/APIConnectionError.md)
- [APIError](https://docs.typesafe.ai/sdk/javascript/api/classes/APIError.md)
- [APIPromise&lt;T&gt;](https://docs.typesafe.ai/sdk/javascript/api/classes/APIPromise.md)
- [APITimeoutError](https://docs.typesafe.ai/sdk/javascript/api/classes/APITimeoutError.md)
- [APIUserAbortError](https://docs.typesafe.ai/sdk/javascript/api/classes/APIUserAbortError.md)
- [AuthenticationError](https://docs.typesafe.ai/sdk/javascript/api/classes/AuthenticationError.md)
- [BadRequestError](https://docs.typesafe.ai/sdk/javascript/api/classes/BadRequestError.md)
- [InternalServerError](https://docs.typesafe.ai/sdk/javascript/api/classes/InternalServerError.md)
- [NotFoundError](https://docs.typesafe.ai/sdk/javascript/api/classes/NotFoundError.md)
- [PermissionDeniedError](https://docs.typesafe.ai/sdk/javascript/api/classes/PermissionDeniedError.md)
- [RateLimitError](https://docs.typesafe.ai/sdk/javascript/api/classes/RateLimitError.md)
- [TypeSafeClient](https://docs.typesafe.ai/sdk/javascript/api/classes/TypeSafeClient.md)
- [TypeSafeError](https://docs.typesafe.ai/sdk/javascript/api/classes/TypeSafeError.md)
- [UnprocessableEntityError](https://docs.typesafe.ai/sdk/javascript/api/classes/UnprocessableEntityError.md)

#### 介面（Interface）

- [ChoiceQuestion&lt;T&gt;](https://docs.typesafe.ai/sdk/javascript/api/interfaces/ChoiceQuestion.md)
- [ChoiceResponse&lt;T&gt;](https://docs.typesafe.ai/sdk/javascript/api/interfaces/ChoiceResponse.md)
- [Logger](https://docs.typesafe.ai/sdk/javascript/api/interfaces/Logger.md)
- [ModelCard](https://docs.typesafe.ai/sdk/javascript/api/interfaces/ModelCard.md)
- [Models](https://docs.typesafe.ai/sdk/javascript/api/interfaces/Models.md)
- [NoulQuestion](https://docs.typesafe.ai/sdk/javascript/api/interfaces/NoulQuestion.md)
- [NoulResponse](https://docs.typesafe.ai/sdk/javascript/api/interfaces/NoulResponse.md)
- [Questions](https://docs.typesafe.ai/sdk/javascript/api/interfaces/Questions.md)
- [RequestOptions](https://docs.typesafe.ai/sdk/javascript/api/interfaces/RequestOptions.md)
- [RetryPolicy](https://docs.typesafe.ai/sdk/javascript/api/interfaces/RetryPolicy.md)
- [ScoreQuestion&lt;T&gt;](https://docs.typesafe.ai/sdk/javascript/api/interfaces/ScoreQuestion.md)
- [ScoreResponse&lt;T&gt;](https://docs.typesafe.ai/sdk/javascript/api/interfaces/ScoreResponse.md)
- [SystemOneRequest&lt;Q&gt;](https://docs.typesafe.ai/sdk/javascript/api/interfaces/SystemOneRequest.md)
- [SystemOneRequestPayload](https://docs.typesafe.ai/sdk/javascript/api/interfaces/SystemOneRequestPayload.md)
- [SystemOneResult&lt;Q&gt;](https://docs.typesafe.ai/sdk/javascript/api/interfaces/SystemOneResult.md)
- [TypeSafeClientConfig](https://docs.typesafe.ai/sdk/javascript/api/interfaces/TypeSafeClientConfig.md)
- [Usage](https://docs.typesafe.ai/sdk/javascript/api/interfaces/Usage.md)
- [WithResponse&lt;T&gt;](https://docs.typesafe.ai/sdk/javascript/api/interfaces/WithResponse.md)

#### 型別別名（Type Alias）

- [ChoiceCriteria](https://docs.typesafe.ai/sdk/javascript/api/type-aliases/ChoiceCriteria.md)
- [Description](https://docs.typesafe.ai/sdk/javascript/api/type-aliases/Description.md)
- [EntryType](https://docs.typesafe.ai/sdk/javascript/api/type-aliases/EntryType.md)
- [EnvVar](https://docs.typesafe.ai/sdk/javascript/api/type-aliases/EnvVar.md)
- [Fetch](https://docs.typesafe.ai/sdk/javascript/api/type-aliases/Fetch.md)
- [JsonValue](https://docs.typesafe.ai/sdk/javascript/api/type-aliases/JsonValue.md)
- [LogLevel](https://docs.typesafe.ai/sdk/javascript/api/type-aliases/LogLevel.md)
- [Question](https://docs.typesafe.ai/sdk/javascript/api/type-aliases/Question.md)
- [ResultFor&lt;T&gt;](https://docs.typesafe.ai/sdk/javascript/api/type-aliases/ResultFor.md)
- [ScoreCriteria](https://docs.typesafe.ai/sdk/javascript/api/type-aliases/ScoreCriteria.md)
- [ScoreLegend&lt;T&gt;](https://docs.typesafe.ai/sdk/javascript/api/type-aliases/ScoreLegend.md)
- [ScoreOf&lt;T&gt;](https://docs.typesafe.ai/sdk/javascript/api/type-aliases/ScoreOf.md)

#### 變數（Variable）

- [ENV](https://docs.typesafe.ai/sdk/javascript/api/variables/ENV.md)
- [LOG_LEVELS](https://docs.typesafe.ai/sdk/javascript/api/variables/LOG_LEVELS.md)
- [VERSION](https://docs.typesafe.ai/sdk/javascript/api/variables/VERSION.md)

#### 函式（Function）

- [choice()](https://docs.typesafe.ai/sdk/javascript/api/functions/choice.md)
- [noul()](https://docs.typesafe.ai/sdk/javascript/api/functions/noul.md)
- [score()](https://docs.typesafe.ai/sdk/javascript/api/functions/score.md)
