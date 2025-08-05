---
title: "Learn Llm"
date: 2025-02-17T17:10:17+08:00
draft: false
---

# pre-training

https://huggingface.co/spaces/HuggingFaceFW/blogpost-fineweb-v1

100000 symbols(tokens)
raw text --->tokenization ---> token

## tokenization

https://tiktokenizer.vercel.app/?model=cl100k_base

## statistic token simulator

1. 这就是所谓的 prediction
2. 参数 各个 token 的权重
3. 模型有大量的知识，存储在上亿的参数之中。这些参数可以视为对超大规模的知识进行的一种有损压缩。超大规模知识的模糊记忆。
4. 按照统计规律给出所谓的答案
5. 模型需要中间结果
6. 概率、统计
7. in-context learning

# post-training

对话集结构 ---> 一维 token 序列
![conversation-structure-to-token-sequence](https://raw.githubusercontent.com/cloudedseal/pictures/main/img/conversation-structure-to-token-sequence.png)

instruct-gpt

强化学习 
reinforcement learning
监督学习
supervised earning
supervised-fine-tuning




# References
1. [视频讲解](https://weibo.com/2194035935/PefWkzTkK#attitude)
2. [https://tiktokenizer.vercel.app/](https://tiktokenizer.vercel.app/)
3. [https://bbycroft.net/llm](https://bbycroft.net/llm)
4. [https://hyperbolic.xyz/](https://hyperbolic.xyz/)
5. [对话数据集](https://atlas.nomic.ai/map/0ce65783-c3a9-40b5-895d-384933f50081/a7b46301-022f-45d8-bbf4-98107eabdbac)
6. [互联网数据集](https://huggingface.co/spaces/HuggingFaceFW/blogpost-fineweb-v1)
