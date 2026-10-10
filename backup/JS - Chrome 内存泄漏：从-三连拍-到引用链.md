> 上周三下午，客服群转来一条用户录屏：后台管理系统开半小时开始掉帧，一小时后标签页直接卡死，只能关掉重开。"刷新就好"——最气人的一句话：说明 bug 100% 可复现，又 100% 藏在看不见的地方。监控一切正常：CPU 不高，接口不慢，JS 报错为零。打开 Chrome 任务管理器盯内存列：100MB、300MB、600MB……**只涨不跌**。得，内存泄漏，前端最经典的慢性病，确诊了。

# Chrome 内存泄漏：从"三连拍"到引用链

# 一、先确认：三条证据，立案再动手

"感觉卡"不等于泄漏。动手前拿证据，免得在错误方向浪费两天。

**证据一：Chrome 任务管理器。** Shift+Esc，按内存排序，做一轮常规操作（切页面、开弹窗）。正常页面内存涨落，泄漏的只涨不跌。

**证据二：JS 堆大小。** 控制台跑：

```javascript
// 每 5 秒打印堆内存（MB）
setInterval(() => {
  const mb = performance.memory.usedJSHeapSize / 1048576;
  console.log(mb.toFixed(1) + ' MB');
}, 5000);
```

操作一轮回来，基线每次抬高一截，就是泄漏。

**证据三：分配时间线。** DevTools → Memory → "Allocation instrumentation on timeline"，记录、操作、停止。**蓝色柱子只涨不跌 = 泄漏实锤**，还能看到"哪次操作"导致上涨，把泄漏和具体动作对应起来。

中两条就立案。记住顺序：**先确认，再动手**。

# 二、找元凶：Heap Snapshot 三连拍，和 Retainers 的读法

确认之后，"三连拍"：

```
1. 切到页面 A，拍 Snapshot 1
2. 切到页面 B，再切回 A，拍 Snapshot 2
3. 切到页面 B，再切回 A，拍 Snapshot 3
4. Snapshot 3 切 "Comparison" 视图，对比 Snapshot 1
```

看 `# New` 列：**不为零的就是泄漏**——页面都卸载了，对象该被回收完，有残留说明有东西抓着不放。为什么拍三张不是两张？第一次切换可能含"缓存预热"的正常增长，三张排除偶然性；Snapshot 3 比 2 还多一截，就是每次切换都在漏。拍之前手动点几次 GC 按钮，把能回收的先回收，剩下的才是真泄漏。

找到泄漏对象，点开看 **Retainers** 面板——"谁引用了它"。GC 回收不了一个对象，永远因为有引用链连着。从下往上读，找到离 GC Root 最近的"那只手"，那就是真凶。读链诀窍：**跳过系统内部引用**（`system / Map` 之类），找你自己代码里的名字——某个组件、闭包、数组。真凶常伪装成叫 `cache` 的数组、叫 `handlers` 的对象、三个月前写的"临时"全局变量。

# 三、三大惯犯，和一个 echarts 的 20MB

十年了，凶手没换过，一直是这三个：

**惯犯一：没 off 的事件监听。** 组件卸载，监听还活在 window 上。重灾区是**全局事件总线**：`mitt`、`eventBus` 的 `on` 注册了没 `off`。SPA 切来切去，每进一次页面注册一批从不注销——SPA 泄漏第一元凶，顺着 Retainers 十次有八次终点是 eventBus。

```javascript
useEffect(() => {
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
// 清理和注册写在同一处，Vue 在 onUnmounted 里清
```

**惯犯二：没 clear 的定时器。** 问题不只是"多跑几秒"，回调闭包抓着整个组件状态——state、props、大对象一个跑不掉。**一个没清的定时器，等于把整个组件钉在内存里。**

**惯犯三：闭包抓着 DOM。** 在 Heap Snapshot 搜 **Detached**——detached DOM 树是铁证。节点本身不大，但连着整个子树、样式、监听，一个 detached 根能拖住几 MB。

```javascript
// 定罪现场：图表组件卸载没调 dispose
useEffect(() => {
  const chart = echarts.init(ref.current);
  chart.setOption(option);
  return () => chart.dispose();  // 第三方库的销毁必须写
}, []);
```

回到开头事故：三连拍显示每次切回数据大屏多出一批 `Detached HTMLCanvasElement` 和 echarts 实例，约 20MB，Retainers 终点是没调 `chart.dispose()` 的图表组件。教训很具体：**第三方库的"销毁"必须和"初始化"写在同一处**——`init` 过的图表、`new` 过的编辑器、`create` 过的播放器，不会自己消失。封装成 hook 让初始化和销毁锁死，靠代码结构不靠记性。

Vue 3 另有特有坑：`watchEffect` 返回的 stop 不调（组件卸载不会自动停）；全局 `reactive` store 里缓存页面级数据（越攒越多）；`v-show` 切换的重型弹窗（display:none 而已，20 次等于 20 个实例全活着，**重型弹窗用 `v-if`**）。

# 四、慢泄漏：最难查的那一种

上面是"快泄漏"——切几次页面几十 MB 没了。真正难的是**慢泄漏**：每次只漏几十 KB，三连拍看不出异常，但页面开 8 小时从 200MB 爬到 1.5GB。换打法：

**拉长时间线**：写脚本循环切换 30 次再对比，单次泄漏小乘以 30 就现形；或 Performance Monitor 录 10 分钟看基线是否缓慢抬头。

**查"只增不减"的集合**：慢泄漏凶手十有八九是只 `push` 从不清理的数组/Map——埋点队列、日志缓存、撤销栈、WebSocket 消息队列。按 Shallow Size 排序找"大得不合理"的数组，点开看内容：

```javascript
// 典型的慢泄漏：只进不出
const eventLog = [];
function track(event) {
  eventLog.push({ ...event, time: Date.now() });
  if (eventLog.length > 1000) eventLog.splice(0, eventLog.length - 1000);
}
```

**警惕"看起来无害"的累积**：开着 DevTools 时 console 打印的大对象会被留着引用（拍 Snapshot 前先清 console）；没关的 Performance 观测；越积越多的 store 历史快照。查慢泄漏就问一句：**这个页面开 8 小时，哪些数据结构会一直变大？**

# 结语

排查公式一句：**Timeline 确认 → Snapshot 对比找 #New → Retainers 找引用链**。预防更短，四个字：**谁创建，谁清理**——useEffect 的 return、onUnmounted、chart.dispose()，是代码和 GC 之间的合同，签了就得履约。

进阶动作：把"三连拍"写成 Puppeteer 脚本（切 10 次页面对比堆基线，涨超 5MB 拦下发布）接进 CI——排查公式给人用，脚本给流程用；ESLint 把 `react-hooks/exhaustive-deps` 开到 error；关键页面上线前跑一遍三连拍，当成和回归测试一样自然的发布检查项。

开头那条录屏，最后定罪的凶手有点好笑：全局 eventBus 上没 off 的监听——每次进页面注册一次从不注销，开半小时攒上千个，回调还抓着整个页面状态。修完，内存曲线终于从"只涨不跌"变成正常涨落。**泄漏和 bug 一样，越早进测试环境发现，修复成本越低**——等用户录屏发到客服群，成本已翻十倍。

##{"timestamp":1774756800}