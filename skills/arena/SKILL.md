---
name: arena
description: "对同一个任务派出 N 个并行候选，选出底座，把落选者最强的部分嫁接进底座。用于 /arena、'arena this'、'throw it in the arena'，或当一件不平凡的产物只试一次就会把错误形态锁死的时候。"
disable-model-invocation: true
---

# Arena（竞技场）

在启动任何东西之前，开一份待办清单，每个阶段一条（框定、扇出、交叉评审、选底座、嫁接、验证）。

## Phase A：框定

每个候选拿到同样的提示词（prompt），所以提示词就是契约。

1. 说清每个候选要产出什么产物。
2. 推导评分标准。针对*这个*任务的成功长什么样，给出 3 到 6 条具体、可打分的标准。它是 Phase D 里挑选者用的工具。候选只看到任务。
3. 从 `arena runners` 行挑选选手，该行在 `~/.cursor/rules/pstack-models.mdc` 里，默认是 `claude-fable-5-2-high`、`claude-opus-5-5-high-fast`、`gemini-4-neon-xhigh`。`auto` 或 `inherit-parent` 条目就是父模型，所以省略 `model`，派 `generalPurpose`。如果 Task 工具拒绝某个 slug，就用同系列里最近的合法 slug，或者去掉该席位，至少保留两个选手，并说明情况。系列按前缀判定，最长匹配优先：`claude-fable-*`、`claude-*`、`gemini-*`，判定前先去掉开头的 `cursor-`。其他 slug 的系列是它第一个连字符之前的文本。多个设计方向就多派。工作靠生成量取胜、不那么吃判断力时，可以让同一个模型重复出场。按 **poteto-mode** 技能的子代理（Subagents）一节，为每个子代理（subagent）挑选推理等级。
4. 给每个候选独立的输出位置（能用 git 工作树就用，否则用 `/tmp/arena-<slug>/candidate-<n>/`），遵照 **separate-before-serializing-shared-state** 原则技能。

## Phase B：扇出

在一条消息里派出全部 N 个子代理，带 `run_in_background: true`。每个都给：任务、共享背景材料的路径、它自己的输出路径，以及产出产物的指令，外加一段简短理由，写明它考虑过哪些替代方案、否掉了什么。

如果有候选没产出，就按 N-1 继续，并记下掉队者。

## Phase C：交叉评审

Phase B 所有候选完成后，从 `arena cross-judge pool` 行派一个只读的评审，该行在 `~/.cursor/rules/pstack-models.mdc` 里，默认是 `claude-opus-5-5-high-fast`、`gemini-4-neon-xhigh`。评审绝不能来自父模型的厂商。做这项检查时 Fable 算作 Claude，所以取另一家厂商的第一个条目。`auto` 或 `inherit-parent` 条目就是父模型。如果你不知道父模型是什么，就用第一个真实 slug，并说明厂商检查没有跑。如果 Task 工具拒绝某个 slug，就用同系列最近的合法 slug，再不行就用列表里的下一个条目，实在不行就跳过评审，并说明情况。你在 Phase D 阅读的同时，评审按路径标签给每个候选在每条评分标准上打分，并推荐一个底座。

## Phase D：选底座

选之前把每个候选从头读到尾。对照评分标准逐条打分，不要凭整体感觉。Phase C 派了评审时，与它对答案。底座选得一致，就确认这个选择。不一致说明你们有一个带偏见，或者评分标准含糊。决定之前把两边的理由都读了。

选那个未来维护者最容易扩展、又不会打破不变量的底座。两个不相上下时，优先选边界更干净或 API 更小的，遵照 Laziness Protocol（懒惰协议）。

## Phase E：嫁接

把每个落选的候选再走一遍，看有什么值得移植到底座里，通常每个候选就一两处，不是大部分。手工把每个嫁接折叠进去，遵照 **redesign-from-first-principles** 原则技能，让结果在同一个心智模型下保持连贯。不要机械粘贴。

如果候选收敛到同一个形态，这是强烈共识。记下来，就发这个共识形态。不需要嫁接。如果它们分歧巨大，说明 Phase A 规格不足。重新框定再跑一轮，不要把分歧平均一下了事。

## Phase F：验证

合成产物按其他输出一样验证，遵照 **prove-it-works** 原则技能。如果验证发现了 arena 漏掉的问题，要么 Phase A 错了（重新框定再跑），要么某个候选发现了它而你漏了嫁接（回到 Phase E）。不要粉饰过去。

## 产出

一份合成产物，附一份简短的合成说明，写明底座是谁、为什么选它、交叉评审的裁定（或 Phase C 跳过了评审）、每个嫁接来自哪个候选、否掉了什么及原因、有没有收敛或掉队、以及验证结果。
