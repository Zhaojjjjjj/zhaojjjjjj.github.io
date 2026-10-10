> Monorepo 里改了一个按钮的样式，CI 跑了 30 分钟——因为它把 10 个包全量 build + 全量 test 了一遍。Nx 和 Turborepo 把 30 分钟压到 5 分钟，靠的不是"更快"，是**"不做"**：没改的代码，一个字节都不碰。真问题是：**机器怎么知道"哪些没改"？**

# Monorepo：5 分钟 CI 背后，是"增量计算"这门学问

Nx 和 Turborepo 是 2026 年 monorepo 的两个主流方案。它们的宣传语都是"快"，但快的来源完全不同——理解这个差异，比选哪个工具重要。

# 一、第一个机制：affected——"改了啥"推导出"影响啥"

```bash
nx affected -t test --base=main
```

这行命令背后是一次**图计算**：

```
1. 建图：扫描所有 package.json / project.json，
   构建"包依赖图"（ui → web，ui → docs，api → web...）

2. 找 diff：git diff main...HEAD，得到"改了哪些文件"

3. 文件 → 包：改了 packages/ui/src/Button.tsx → 属于 ui 包

4. 图上传播：ui 变了 → 依赖 ui 的 web、docs 都"受影响"
   → 只 test [ui, web, docs]，跳过 api 和其他 6 个包
```

关键洞察：**affected 不是"文件级"的，是"包级 + 依赖传播"的**。改了 ui 的样式，web 的快照测试可能挂——即使 web 的代码一个字没动。**依赖图的传递闭包，才是"受影响"的准确定义。** 这也是手写脚本替代不了 Nx 的原因：`git diff --name-only` 只能告诉你"改了哪些文件"，算不出"这些文件会炸掉哪些包"。

Turborepo 的 `--filter=...[HEAD]` 是简化版：只看"改了的包"，不做完整的依赖传播。**小 monorepo（<50 包）够用，大 monorepo 会漏**——这就是两者分化的起点。

# 二、第二个机制：内容寻址缓存——"算过就别再算"

```json
// turbo.json
{
  "tasks": {
    "build": { "outputs": ["dist/**"] }
  }
}
```

Turborepo 的缓存逻辑：

```python
# 伪代码：任务指纹
def fingerprint(task, package):
    inputs = hash(
        source_files(package),      # 源码内容
        dependencies(package),      # 依赖的版本（lockfile）
        env_vars(task),             # 环境变量
        task_command                 # 命令本身
    )
    return inputs

# 执行前查缓存
if cache.has(fingerprint("build", "ui")):
    return cache.get(...)   # 直接解压 dist/，1 秒
else:
    run_build()             # 真跑，5 分钟，然后存缓存
```

**这是"内容寻址"，不是"时间戳"**：只要输入的每一个字节都没变，输出一定没变——直接复用。远程缓存（Vercel Remote Cache / Nx Cloud）把这个逻辑扩展到全团队：**你同事算过的，你不用再算**。

这里有个微妙的设计：指纹必须包含**环境变量和命令本身**。`NODE_ENV=production` 和 `NODE_ENV=development` 的 build 产物不同——如果指纹漏了环境变量，缓存会返回"错的正确结果"。**缓存的正确性，取决于指纹的完备性**，这是所有增量系统最容易翻车的地方。

# 三、为什么"不做"比"快"重要 10 倍

```
全量：  10 个包 × (build 1.5min + test 2min) = 35 分钟
增量：  改 1 个包 → affected 算出 3 个包 → 其中 2 个命中缓存
       = 1 个包真跑 (3.5min) + 2 个包解压缓存 (10s) ≈ 4 分钟
```

**9 倍提速里，"更快"贡献了 0 倍**——机器没变快，只是 90% 的工作被证明"不用做"。这是 monorepo 工具和"换更快的 CI 机器"的本质区别：**前者是算法胜利（O(n) → O(affected)），后者是 brute force。**

# 四、选型：看规模，不看功能

```
<50 包，JS 全栈，小团队 → Turborepo
  理由：JSON 配置 10 分钟上手，affected 简化版够用，
        Vercel 部署无缝

50+ 包，多语言，大团队 → Nx
  理由：完整的依赖图传播、50+ 插件、1000 包也扛得住
  代价：学习曲线陡，配置复杂
```

**经验法则的反面**：已经在用的别换。Turborepo 迁 Nx（或反向）的迁移成本，远大于两者之间的性能差——**工具选型的第一原则是"别折腾"，第二才是"快"。**

# 结语

Monorepo 工具的故事可以浓缩成一句话：

> **CI 优化的终极形态不是"跑得快"，是"证明不用跑"。affected 用依赖图证明"哪些包不用测"，内容寻址用哈希证明"哪些任务不用算"。**

Nx 和 Turborepo 只是这句话的两个实现。记住这个顺序：**先增量（affected + 缓存），再并行（分片），最后才是加机器**——颠倒顺序，就是在烧钱。

##{"timestamp":1787976000}