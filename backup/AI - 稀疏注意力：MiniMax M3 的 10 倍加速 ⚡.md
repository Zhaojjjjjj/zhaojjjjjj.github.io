> 2026 年 5 月，MiniMax 预告 M3：自研 MSA 稀疏注意力，Prefill 提速 9.7 倍、Decoding 提速 15.6 倍。注意力是 Transformer 最贵的部分，稀疏化是必经之路。这篇讲讲稀疏注意力。

## 注意力为什么贵

```
标准注意力：O(n²)
n=100K tokens → 100亿次运算
```

**平方复杂度**是长上下文的第一杀手。100K 上下文，注意力占 90% 的计算。

## 稀疏化的直觉

```
 dense：每个 token 看所有 token（100K × 100K）
sparse：每个 token 只看"重要的"（100K × 1K）
```

**注意力矩阵是稀疏的**——大部分 token 对之间的相关性接近 0。Dense 计算了 100 亿次，其中 99 亿是浪费。

## 稀疏三式

### 1. 滑动窗口（Sliding Window）

```
每个 token 只看前后各 512 个
O(n × w)，w=1024
```

Mistral 2023 年的做法。**局部性假设**：离得近的更相关。简单有效，但长程依赖会丢。

### 2. 稀疏模式（Sparse Pattern）

```
BigBird：窗口 + 随机 + 全局
→ 理论证明：稀疏也能逼近 dense
```

2020-2022 年的学术路线，**复杂，工程化难**。

### 3. MSA（MiniMax Sparse Attention）

MiniMax M3 的自研方案（从预告推断）：

```
动态稀疏：模型自己学"该看谁"
→ 不是固定模式，是 content-based
Prefill 9.7x，Decoding 15.6x
```

**动态 > 静态**：固定的稀疏模式会丢信息，让模型自己决定"谁重要"，效果更好。

## 工程实现

```python
# 朴素：dense
attn = softmax(Q @ K.T / sqrt(d)) @ V  # O(n²)

# 稀疏：只算 top-k
scores = Q @ K.T / sqrt(d)
topk_idx = scores.topk(k=1024, dim=-1).indices  # 只取最重要的 1024 个
attn = sparse_softmax(scores, topk_idx) @ V  # O(n×k)
```

**难点**：topk 本身要 O(n²) 的 score 计算——**用近似**（LSH、聚类）先粗筛，再精算。

## 一句话总结

稀疏注意力是"**注意力的瘦身**"：从 O(n²) 到 O(n×k)。2026 年，长上下文的竞争从"窗口多大"转向"注意力多稀疏"。记住：**模型不需要看所有，只需要看对的**——而"对的"，可以学出来。

##{"timestamp":1779422400}