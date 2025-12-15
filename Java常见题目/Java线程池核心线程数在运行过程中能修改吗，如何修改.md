## Java 线程池核心线程数在运行过程中能修改吗？如何修改？

这是一个**Java 并发面试常考题**，核心是理解 **线程池核心线程数的概念**以及 **如何动态调整**。我帮你整理成 **概念 → 可否修改 → 修改方式 → 面试总结**。

## 面试总结一句话

> **Java 线程池的核心线程数可以在运行过程中通过 `ThreadPoolExecutor.setCorePoolSize()` 动态修改，线程池会根据新设置调整线程创建和回收策略。**

------

## 一、核心概念

### 1️⃣ 核心线程数（corePoolSize）

- 定义：线程池中**长期保留的线程数**，即使空闲也不会被回收（除非 `allowCoreThreadTimeOut=true`）
- 作用：
  - 核心线程用于处理任务队列中的任务
  - 当任务量大于核心线程数时，会创建非核心线程，非核心线程空闲超过 `keepAliveTime` 会被回收

### 2️⃣ 线程池类型

- `ThreadPoolExecutor` 提供动态调整核心线程数的能力
- 通过 `Executors` 工厂方法创建的线程池（如 `newFixedThreadPool`）底层也是 `ThreadPoolExecutor`

------

## 二、核心线程数能否修改？

✅ **可以动态修改核心线程数**，核心线程数不是固定的。

> 注意：调整核心线程数不会立即杀掉多余线程，线程池会根据新设置动态调整。

------

## 三、修改方法

### 1️⃣ 使用 `ThreadPoolExecutor.setCorePoolSize(int corePoolSize)`

```
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5,       // corePoolSize
    10,      // maximumPoolSize
    60, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>()
);

// 修改核心线程数
executor.setCorePoolSize(8);
```

### 2️⃣ 注意事项

1. **新核心线程数大于旧核心线程数**
   - 当有新任务提交，线程池会立即创建更多核心线程
2. **新核心线程数小于旧核心线程数**
   - 已经存在的多余核心线程不会立即停止
   - 空闲时被回收
3. **`allowCoreThreadTimeOut`**
   - 如果允许核心线程超时，核心线程数减少后会快速回收

```
executor.allowCoreThreadTimeOut(true);
```

------

## 五、面试可能追问

- ❓ 核心线程数与最大线程数的关系？
  - 核心线程数 ≤ 最大线程数
  - 当任务数 > 核心线程数 → 创建非核心线程
- ❓ 调整核心线程数会立即销毁线程吗？
  - ❌ 不会，空闲时才回收，除非 `allowCoreThreadTimeOut=true`
- ❓ `ScheduledThreadPoolExecutor` 可以修改核心线程数吗？
  - ✅ 可以，同样继承自 `ThreadPoolExecutor`