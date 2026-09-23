# TypeSafe Agent Skills

用於使用 [TypeSafe](https://typesafe.ai) 建置 Agent 的技能：從 System One 模型取得具型別的判斷與機率。

你可以在 GitHub 上閱讀 [SKILL.md](https://github.com/typesafe-ai/skills/blob/main/skills/typesafe-ai/SKILL.md)，或取得其 [原始 Markdown](https://raw.githubusercontent.com/typesafe-ai/skills/main/skills/typesafe-ai/SKILL.md)。

## 安裝

### Claude Code 外掛

在終端機執行：

```bash
claude plugin marketplace add typesafe-ai/skills
claude plugin install typesafe@typesafe-ai
```

### 透過 skills.sh 安裝到其他 Agent

```bash
npx skills add typesafe-ai/skills --skill typesafe-ai
```

請在提示時選擇你的 Agent。預設會以專案為範圍安裝；若要全域安裝，請加上 `-g`。

如需提示詞、手動安裝方式與更新說明，請參閱[安裝指南](https://docs.typesafe.ai/agent-skill#installation)。

## 使用方式

例如，可以這樣要求你的 Agent：

> 使用 TypeSafe，依部門分流傳入的客服工單，並將不確定的判斷交由人工審核。

在 Claude Code 中，也可以使用 `/typesafe:typesafe-ai` 明確呼叫此外掛技能。

| 技能 | 用途 |
|---|---|
| [typesafe-ai](skills/typesafe-ai/SKILL.md) | 設計 TypeSafe 工作流程、查找最新文件與操作手冊，並在程式碼中組合具型別的判斷 |

## 文件

[docs.typesafe.ai](https://docs.typesafe.ai/introduction)  
[Playground](https://console.typesafe.ai/playground)  

## 授權條款

[MIT](LICENSE)。
