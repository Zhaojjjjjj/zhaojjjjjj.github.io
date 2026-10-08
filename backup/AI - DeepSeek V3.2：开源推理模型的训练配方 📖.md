> DeepSeek 在 2025 年 9 月放出了 V3.2-Exp 实验版。作为"最会写技术报告"的公司，DeepSeek 每次发布都附带干货。这篇结合 V3 系列的公开信息，聊聊开源推理模型的"训练配方"。

## V3 系列的配方

DeepSeek V3（2024 年底）的报告曾震惊行业：**只用 557 万美元训出 671B 模型**。配方有三味主药：

1. **FP8 混合精度训练**：业界第一次大规模用 FP8 训大模型。通信量减半，速度翻倍。代价是数值稳定性——DeepSeek 写了整整一章讲怎么防止 loss spike。
2. **Multi-head Latent Attention（MLA）**：把 KV Cache 压缩到极致。推理时 KV 占用只有 MHA 的 1/10——这是 DeepSeek API 能卖得比谁都便宜的根本原因。
3. **DeepSeekMoE + 无辅助损失**：MoE 不用 aux loss 做负载均衡，靠"偏置项"动态调。这是反常识的，但work。

## V3.2 的增量：强化学习

V3.2-Exp（含 Thinking 版）的重点是 **RL 后训练**：

```
SFT（学格式）→ RL（学推理）
```

DeepSeek 的 RL 和别人不一样：**不用 PPO，用 GRPO**（Group Relative Policy Optimization）。不用 critic 模型，省一半显存，效果还更好。R1 就是这么训出来的，V3.2 是把这套搬到 V3 基座上。

## 开源 vs 闭源的训练差异

| 维度 | DeepSeek（开源） | OpenAI/Anthropic（闭源） |
|---|---|---|
| 论文 | 发详细技术报告 | 发博客，不发细节 |
| 复现 | 社区一周复现 | 只能猜 |
| 优化目标 | 极致性价比 | 极致性能 |
| 数据 | 公开配方思路 | 黑盒 |

DeepSeek 的策略是"**用 openness 换生态**"：报告越详细，用的人越多，反馈越多，下一代越强。这是阳谋。

## 给开发者的启示

1. **FP8 训练**：如果你的卡是 H100/H200，别再用 BF16 了，FP8 省一半钱。
2. **MLA 的推理红利**：部署 DeepSeek 系模型，KV Cache 小=batch 可以开大=QPS 高。
3. **GRPO**：做 RLHF/RLVR 时，GRPO 比 PPO 省资源，先试 GRPO。

## 一句话总结

DeepSeek 的配方是"工程极致主义"：FP8、MLA、GRPO，每一个都是把成本砍半的硬功夫。V3.2-Exp 证明，开源模型在推理能力上已经摸到了闭源的屁股——而它的报告，是全行业最好的免费教材。

##{"timestamp":1758513600}