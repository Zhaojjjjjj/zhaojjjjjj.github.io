> `Type 'string' is not assignable to type 'never'`——第一次见到这句报错的人，都会愣住：我明明写的是 string，它从哪变出个 never？真问题是：**你以为你在"写类型"，其实类型检查器在"猜类型"**——而 `never` 就是它猜错时的默认答案。

# TS 类型报错：`never` 是类型检查器"猜错"时的样子

TypeScript 的类型系统有一半是"推断"：你不写注解，它就猜。`never` 报错的本质，是**推断的起点错了**——而起点错一次，后面全错。

# 一、`never[]`：最常见的误推断

```typescript
const arr = [];           // 你没写类型，TS 开始猜
arr.push("hello");        // Error: Argument of type 'string'
                          //   is not assignable to parameter of type 'never'
```

发生了什么？TS 的推断是**从字面量出发、只向前看**的：

```
第 1 行：const arr = []
  → 字面量 [] 里没有任何元素
  → 没有任何信息能推断元素类型
  → 退化成 never[]（"空到不可能有元素"的数组）

第 2 行：arr.push("hello")
  → arr 是 never[]，push 期望 never
  → string ≠ never，报错
```

关键洞察：**TS 的推断不看"未来"**。它在第 1 行做决定时，看不到第 2 行的 `push("hello")`。这不是 bug，是设计——类型检查器是单遍的，它必须在看到声明的那一刻就定下类型。`never` 就是"信息为零时的默认类型"：**一个连一个例子都没见过的类型，只能是"不可能存在"的类型**。

修复不是"修报错"，是"给信息"：

```typescript
const arr: string[] = [];   // 直接告诉它答案，别让它猜
```

# 二、`unknown`：和 `any` 的一字之差

```typescript
declare const data: unknown;

data.foo;   // Error: Object is of type 'unknown'
```

`unknown` 是 TS 类型系统的"顶类型"（top type）——**所有类型都是它的子类型**。这句话反过来说：`unknown` 的值，你对它**一无所知**，所以什么操作都不允许。

对比 `any`：

```
any：    "我放弃检查" —— 底层是把类型检查器关掉
unknown： "我要求证明" —— 检查器开着，但你要先用 typeof/instanceof
          把 unknown 收窄成具体类型，才能用
```

```typescript
if (typeof data === "object" && data !== null) {
    (data as { foo: string }).foo;   // 先证明，再使用
}
```

设计哲学：`any` 是 2012 年的妥协（"先让 JS 转过来再说"），`unknown` 是 2020 年的修正（"不知道就别用"）。TS 7.0 把 `strict` 默认开启后，`any` 会成片爆警告——**`unknown` 不是 `any` 的替代品，它是 `any` 的还债。**

# 三、读报错的正确姿势：从"期望"倒推

TS 报错的格式是固定的，读法也是：

```
Type 'X' is not assignable to type 'Y'.
  The expected type comes from property 'data'
  which is declared here on type 'Props'
```

**从下往上读**：先看"Y 是在哪被期望的"（声明点），再看"X 是从哪来的"（赋值点）。90% 的报错，真相是这两处**对同一个东西有不同的假设**：

```typescript
// 声明点：我以为是这个
interface Props { data: string[] }

// 赋值点：实际给了这个
<Comp data={maybeUndefined} />   // string[] | undefined ≠ string[]
```

而最高效的调试工具不是读报错，是**悬停**：把鼠标放在变量上，看 TS 推断出的类型。报错说的是"结果"，悬停给你看的是"过程"——**类型不匹配的根因，永远在推断链条的起点，不在报错的那一行。**

# 四、一个心智模型：把类型检查器当成"严格的会计"

```
会计原则：每一笔账，都要有凭证

const arr = []           → 没有凭证（空数组），记为 never[]
arr.push("hello")        → 凭证来了，但账已经记死了，对不上

const arr: string[] = [] → 你亲手写了凭证，账从一开始就是对的
```

`never` 是会计的"坏账标记"：**当某一笔的类型实在无从得知，记成 never——之后任何"有类型"的东西往里放，都会触发警报。** 理解了这一点，`never` 报错就从"天书"变成了"线索"：它在告诉你——**往前找，找到那个"信息为零"的起点。**

# 结语

TS 类型报错的排查，只有一条心法：

> **报错的行不是病因，是症状。沿着推断链往前走，找到那个"检查器被迫猜"的地方——`never` 永远站在起点等你。**

而 `unknown` 则是另一面：当你真的不知道类型时，诚实地写 `unknown`，然后用收窄去证明。7.0 之后，`any` 是债，`unknown` 是路。

##{"timestamp":1785297600}