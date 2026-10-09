> 页面开久了，从 100MB 涨到 1GB，最后卡死。前端内存泄漏比后端更隐蔽——没有 OOM killer，只有用户的抱怨。这篇讲讲用 Chrome DevTools 抓前端内存泄漏。

## 症状

- SPA 切几个页面，内存只涨不跌。
- `performance.memory.usedJSHeapSize` 持续上升。
- 用户说"用一会儿就卡"，刷新就好。

## 排查三步

### 1. 确认泄漏：时间线

```
DevTools → Memory → 选 "Allocation instrumentation on timeline"
→ 操作（切页面几次）→ 停止
```

**蓝色柱子只涨不跌** = 泄漏。先确认，再动手。

### 2. 找元凶：Heap Snapshot 对比

```
1. 切到页面 A，拍 Snapshot 1
2. 切到页面 B，再切回 A，拍 Snapshot 2
3. 切到页面 B，再切回 A，拍 Snapshot 3
4. 对比 Snapshot 3 vs Snapshot 1，用 "Comparison" 视图
```

**`# New` 不为零的就是泄漏**。正常情况，切回来应该和第一次一样。

### 3. 看 Retainers：谁抓着不放

点开泄漏的对象，看 **Retainers** 面板——**是谁引用了它，导致 GC 回收不了**。

## 三大惯犯

### 1. 没清理的事件监听

```javascript
// ❌ 组件卸载了，监听还在
useEffect(() => {
  window.addEventListener('resize', handleResize);
}, []);

// ✅ 清理
useEffect(() => {
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```

**Vue 同理**：`onUnmounted` 里清。`mitt`/`eventBus` 的 `on` 没 `off`，是 SPA 泄漏第一元凶。

### 2. 没清的定时器

```javascript
// ❌ setInterval 一直跑，闭包抓着组件状态
useEffect(() => {
  const id = setInterval(() => setCount(c => c + 1), 1000);
  return () => clearInterval(id);  // 别忘了这行
}, []);
```

### 3. 闭包抓着 DOM

```javascript
// ❌ detached DOM：从文档移除了，但 JS 还引用着
const cache = [];
function addEl() {
  const div = document.createElement('div');
  document.body.appendChild(div);
  cache.push(div);  // 即使 removeChild 了，cache 还抓着
}
```

DevTools 里搜 **Detached**——detached DOM 树是泄漏的铁证。

## 防患

1. **ESLint 规则**：`react-hooks/exhaustive-deps` 开到 error。
2. **组件卸载测试**：挂载→卸载→挂载，heap 应该回到基线。写进 e2e。
3. **WeakMap/WeakRef**：缓存 DOM 用 `WeakMap`，不阻止 GC。

## 一句话总结

前端内存泄漏排查公式：**Timeline 确认 → Snapshot 对比找 #New → Retainers 找引用链**。三大惯犯：没 off 的监听、没 clear 的定时器、被闭包抓着的 DOM。记住：**谁创建，谁清理**——useEffect 的 return、onUnmounted，不是摆设。

##{"timestamp":1774756800}