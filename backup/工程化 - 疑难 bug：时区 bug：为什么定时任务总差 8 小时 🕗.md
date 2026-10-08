> 定时任务设的是"每天凌晨 2 点跑"，结果它每天 10 点跑。或者：夏令时那天，任务跑了两次（一次没跑）。时区 bug 是"最熟悉的陌生人"——这篇一次讲透。

## 症状

```javascript
// 需求：每天北京时间凌晨 2 点跑
cron.schedule('0 2 * * *', task);
// 结果：每天 UTC 2 点跑 = 北京时间 10 点
```

## 根因一：cron 的时区

大多数 cron 库**默认用系统时区**（服务器通常是 UTC），不是你想要的时区。

```javascript
// 修复：显式指定时区
cron.schedule('0 2 * * *', task, {
  timezone: 'Asia/Shanghai'  // node-cron 支持
});

// bullmq / agenda 同理，都有 timezone 选项
```

**铁律**：任何定时任务，时区必须显式声明，永远不要依赖"默认"。

## 根因二：Date 的坑

```javascript
new Date('2025-11-29')        // UTC 午夜！不是本地午夜
new Date('2025-11-29T02:00') // 本地时间（无 Z 后缀）
new Date('2025/11/29')       // 本地时间（Safari 甚至解析不了 - 格式）

task.runAt = new Date('2025-11-29')  // 存的是 UTC，取出来差 8 小时
```

**三条军规**：

1. **存储用 UTC**：数据库里永远存 UTC 时间戳，显示时再转本地。
2. **传输用 ISO8601 带时区**：`2025-11-29T02:00:00+08:00`，别传裸字符串。
3. **计算用库**：`dayjs`/`date-fns` + `utc`/`timezone` 插件，别手算 `+8*3600*1000`（夏令时会死）。

## 根因三：夏令时（DST）

```
美国：2025 年 3 月 9 日凌晨 2 点 → 直接跳到 3 点（2:00-2:59 不存在）
     2025 年 11 月 2 日凌晨 2 点 → 出现两次（2:00-2:59 有两遍）
```

- 设在 2:30 的任务，3 月 9 日**不会跑**（那一小时不存在）。
- 设在 1:30 的任务，11 月 2 日**跑两次**。

**解法**：跨国业务，定时任务用 UTC 时间，别用本地时间。UTC 没有夏令时。

## 终极方案

```javascript
// 1. 服务器统一 UTC
// 2. 任务调度用 UTC cron
// 3. 只在"展示给用户"时转本地时区
// 4. 用 dayjs 处理所有转换

const dayjs = require('dayjs');
const utc = require('dayjs/plugin/utc');
const timezone = require('dayjs/plugin/timezone');
dayjs.extend(utc); dayjs.extend(timezone);

// 存
db.save({ runAt: dayjs().utc().format() });
// 显示
dayjs.utc(runAt).tz('Asia/Shanghai').format('YYYY-MM-DD HH:mm');
```

## 一句话总结

时区 bug 的本质是"**隐式假设**"：假设服务器是本地时区、假设字符串是本地时间、假设一天永远 24 小时。根治就一条——**UTC 存、UTC 算、显示再转**。下次定时任务差 8 小时，先看三处：cron 时区、Date 解析、夏令时。

##{"timestamp":1764388800}