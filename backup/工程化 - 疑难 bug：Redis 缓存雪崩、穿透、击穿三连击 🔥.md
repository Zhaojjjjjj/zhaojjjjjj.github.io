> 缓存雪崩、穿透、击穿——Redis 三大经典故障，名字像绕口令，但每个都能让生产环境跪一次。这篇一次讲透，附防御代码。

## 雪崩：缓存集体过期

```
场景：首页的 10 万个商品缓存，TTL 都是 1 小时，整点一起过期
结果：10 万请求同时打到 MySQL，DB 直接被打死
```

**本质**：过期时间太整齐。

### 防御

```python
# 1. TTL 加随机抖动——最简单，最有效
ttl = 3600 + random.randint(-600, 600)

# 2. 热点数据永不过期（逻辑过期）
# 缓存里存 {data, expire_at}，过期后异步刷新，请求永远有数据可返回
value = redis.get(key)
data, expire_at = json.loads(value)
if time.now() > expire_at:
    # 异步刷新，不阻塞请求
    asyncio.create_task(refresh_cache(key))
return data
```

## 穿透：查不存在的数据

```
场景：有人用脚本刷 /user/999999999（不存在的用户）
结果：缓存没有，DB 也没有，但每个请求都要查一次 DB
```

**本质**：缓存对"不存在"没有记忆。

### 防御

```python
# 1. 缓存空值（最常用）
user = db.query(id)
if user is None:
    redis.setex(f"user:{id}", 60, "NULL")  # 空值也缓存 1 分钟
    return None

# 2. 布隆过滤器（终极方案）
# 10 亿用户 ID，布隆过滤器只要 ~1GB 内存，误判率 1%
if not bloom.might_contain(id):
    return None  # 一定不存在，连缓存都不查
```

## 击穿：热点 key 过期瞬间

```
场景：某明星的微博，缓存 1 分钟过期
结果：过期那 1 秒，10 万请求同时发现缓存没了，同时去 DB 查
```

**本质**：雪崩是"很多 key 一起过期"，击穿是"一个热点 key 过期"。

### 防御

```python
# 互斥锁：只让一个请求去 DB，其他等
lock = redis.lock(f"lock:{key}", timeout=10)
if redis.get(key) is None:
    if lock.acquire(blocking=False):  # 抢到锁的去查 DB
        try:
            data = db.query(key)
            redis.setex(key, 60, data)
        finally:
            lock.release()
    else:  # 没抢到的等 50ms 后重试（从缓存读）
        time.sleep(0.05)
        return redis.get(key)
```

## 三者的区别（一张表）

|  | 雪崩 | 穿透 | 击穿 |
|---|---|---|---|
| 对象 | 大量 key | 不存在的 key | 单个热点 key |
| 原因 | TTL 整齐过期 | 无"不存在"记忆 | 热点过期瞬间 |
| 解法 | TTL 抖动/逻辑过期 | 空值缓存/布隆 | 互斥锁 |

## 一句话总结

缓存三连击的本质都是"DB 被意外打到"。防御思想就一句：**永远不要让 DB 直接面对流量洪峰**——抖动 TTL 削峰、空值缓存设防、互斥锁限流。Redis 是盾，但盾也要有盾的用法。

##{"timestamp":1759118400}