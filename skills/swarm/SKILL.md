---
name: swarm
description: "扇出 N 个并行的工作单元，把它们收干，返回一份报告。用于 /swarm、'swarm this'，或并行覆盖、竞速、车轮战和探索。"
disable-model-invocation: true
---

# Swarm（蜂群）

## 开始

在启动任何东西之前，开一份待办清单，每个阶段一条。

1. 框定
2. 扇出
3. 汇总
4. 报告

## Phase A：框定

1. 说清完成谓词，以及蜂群必须交回的产物或报告。
2. 选形态。切成切片分区、让 N 个工作单元在相同任务书下竞速，或者两者混用。竞速或混合形态要在派出前声明 `first pass`、`rank all` 或 `best-of`。
3. 从 `swarm workers` 行挑工作单元用的模型，该行在 `~/.cursor/rules/pstack-models.mdc` 里。规则或该行缺失，就用 `claude-opus-5-5-high-fast`。`auto` 或 `inherit-parent` 就省略 `model`，让工作单元跑在父模型上。如果 Task 工具拒绝某个 slug，就用默认值并说明情况。它连默认值也拒绝，就从它的报错信息里挑同系列最近的合法 slug。模型竞速要事先点名每一路的模型。按 **poteto-mode** 技能的子代理（Subagents）一节，按任务为每个子代理（subagent）挑选推理等级。
4. 工作单元要写文件时，给它自己可写的输出位置。工作单元验证或测量提交时，每份任务书写明确切的 SHA。测量任务书还要写明方法（样本数、一个样本是什么、顺序）。工作单元把这两样都记进结果里。

## Phase B：扇出

在一条消息里派出全部 N 个工作单元，带 `subagent_type: generalPurpose`、`environment: "cloud"`、`run_in_background: true`，模型按第 3 步，`auto` 或 `inherit-parent` 则不设。只有工作单元需要访问用户电脑上的东西时才用 `environment: "local"`。

工作单元必须从非默认的已推送分支起步时，传 `cloud_base_branch`。

每份任务书都要自成一体。写明目标、范围、确切的切片或竞速路数、怎么验证、报告什么。报告用 `PASS`、`ISSUES` 或 `BLOCKED`，附证据。能证明缺陷的工作单元报告 `ISSUES`，列出它能证明的每一个问题，不只是第一个。

有工作单元掉队，就按 N-1 继续，并记下来。

## Phase C：汇总

读最终结果。没按任务书记下 SHA 和方法的结果丢弃，重新派一次那个工作单元。第二次还不行，就记为缺口。缺口不算通过。覆盖形态下，每个必需的切片都要有结果。竞速形态下，套用事先声明的选取规则。用 first pass、rank all 或 best-of。不要粘贴工作单元的原始倾倒输出。

## Phase D：报告

在对话里返回一份合并报告，带结果表、问题一句话列表、缺口或掉队，用了竞速的还要写竞速规则。
