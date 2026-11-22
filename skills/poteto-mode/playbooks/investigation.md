### Investigation（调查）

**你拥有答案。计划、路由、成文。**

调查请求是只读的。它们产出带引用的解释或建议，不是代码改动。

1. 追踪问题涉及的代码。动机类问题还要走一遍 **why** 技能。
2. 吞吐检查点只留一行：`throughput checkpoint: n/a, read-only investigation`。
3. 产出成果（Overview / Key Concepts / How It Works / Where Things Live / Gotchas），或者当请求是在备选之间做决定时，给出带取舍表的建议。
4. 对回复应用 **unslop** 技能。

不建 PR、不 babysit、不 `architect`，除非这次调查是一个代码改动的前置。如果是，交还给用户，改路由到 Bug fix 或 Feature。

**回复：** 调查成果。对于"我们确定吗？"这类回答，附上你真实的判断和理由。前提错了就顶回去（见自主性（Autonomy）一节）。
