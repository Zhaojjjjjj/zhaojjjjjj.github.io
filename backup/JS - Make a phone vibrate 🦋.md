### 使用 JavaScript 让手机振动

#### Vibration API 简介
Vibration API 是现代 Web 浏览器中的一个功能，可以控制设备的振动功能。主要用于移动设备，大多数桌面设备不具备振动功能。

该 API 核心方法为 `navigator.vibrate()`，用于触发设备振动指定的时间。

#### Vibration API 的基本用法
```javascript
navigator.vibrate(pattern);
```
其中 `pattern` 可以是：
- 一个数字，表示振动持续的毫秒数。
- 一个数组，奇数索引表示振动持续时间，偶数索引表示暂停时间。

例如：
```javascript
// 振动 500 毫秒
navigator.vibrate(500);

// 振动 200 毫秒，暂停 100 毫秒，再振动 200 毫秒
navigator.vibrate([200, 100, 200]);
```

#### 实例
**按钮点击触发振动**

点击按钮时设备会振动 300 毫秒：
```html
<button onclick="navigator.vibrate(300)">振动</button>
```

**模式化振动**

通过数组可以定义复杂的振动模式，如振动 100 毫秒，暂停 50 毫秒，重复多次：
```html
<button onclick="navigator.vibrate([100, 50, 100, 50, 300])">振动模式</button>
```

**停止振动**

调用 `vibrate(0)` 或传入空数组可以停止当前的振动：
```javascript
navigator.vibrate(0);
navigator.vibrate([]);
```

#### 检查浏览器支持
在使用 Vibration API 之前，建议先检查设备或浏览器是否支持：
```javascript
if ("vibrate" in navigator) {
    console.log("支持 Vibration API");
} else {
    console.log("不支持 Vibration API");
}
```

#### 实际应用场景
- **通知**：当收到 Web 应用通知时，触发短暂振动。
- **游戏**：在与游戏元素交互时，通过振动增强用户体验。
- **警报**：使用特定的振动模式提醒用户关键更新或警告。

#### 注意事项与最佳实践
- **电池消耗**：频繁或长时间的振动会迅速耗尽设备电池，建议适量使用。
- **用户体验**：过多的振动可能会令人烦躁，最好提供关闭振动的选项。
- **无障碍功能**：一些用户依赖振动作为无障碍设置的一部分，确保你的应用尊重这些设置。