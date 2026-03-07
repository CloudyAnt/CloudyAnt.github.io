---
title: Prompt Engineering
date: 2026-03-07 17:24:31
categories: Prompt, AI
---

## 概念概述

1. Zero Shot 直接提问
2. Few Shot 给出问答示例，然后提问
3. Chain-of-Thought Prompting 思考链提示。ZeroShot-CoT 是添加一行 "Let's think step by step"，FewShot-CoT 是在示例回答中添加思考步骤
4. Auto CoT。阶段一，模型将数据集中的问题按语义进行分组，从每组中采样代表性问题，使用 ZeroShot-CoT 为挑选的问题生成推理路径。阶段二，从示例库中挑选问题作为 FewShot，可添加 ZeroShot-CoT
5. Meta Prompting 元提示。结构化输入，引导结构化输出
6. Self Consistency 自我一致性。多次输出，少数服从多数。用于提高结果准确度
7. Generated Knowledge Prompting  先让模型生成知识，再回答问题。前提是假设模型内部已有相关知识
8. Prompt Chaining 提示链。拆解任务，将前面任务输出作为后面任务的 Prompt 的输入
9. Tree of Thoughts 思考树。不同于 CoT，对于每一步输出都要求给出多个输出，选择正确输出并继续
10. Retrieval Augmented Generation。向量检索相关内容并作为上下文传给模型
11. Automatic Reasoning and Tool-use (ART)。类似于 Auto CoT，不同的是，它问题检索范围较为精确，而且强调工具调用能力
12. Automatic Prompt Engineer (APE)。模型遵循生成-评估-迭代闭环来自主更新优化 Prompt。多发生在离线优化时，也可以在 pre-response 时优化用户输入，以及在 post-response 后验证输出并迭代
13. Active-Prompt。让模型挑出最难或不确定的问题，由人类进行标注。用以提升复杂、垂直领域的准确率
14. Directional Stimulus Prompting (DSP)。 先生成暗示（Hint/Stimulus），再用这个暗示来引导模型生成最终结果。 
15. PAL (Program-Aided Language Models)。类似于 CoT，但是解决步骤是以编程语言形式展示，在模型输出后，需要调用对应的运行时计算出结果
16. ReAct Prompting。模型按思考-行动-观察循环逻辑工作，强调外部交互。为 ART 提供了逻辑框架
17. Reflexion 反思。Actor 执行，Evaluator 打分或检测，有问题则交给 Self-Reflection 重新审视并在下轮优化
18. Multimodal CoT Prompting 多模态思维链提示。将 CoT 拓展到了多模态，生成思维链时要结合多模态信息