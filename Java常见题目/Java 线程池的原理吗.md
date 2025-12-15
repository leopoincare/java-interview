## Java 线程池的原理

当然，这是 **Java 并发面试的核心题**，重点不在“会不会用”，而在 **是否理解线程池的整体工作机制**。我按 **核心原理 → 执行流程 → 关键参数 → 拒绝策略 → 面试总结** 给你系统讲一遍。

------

## 面试总结一句话

> **Java 线程池基于 ThreadPoolExecutor 实现，通过核心线程、任务队列和最大线程数协同工作，按“先核心、再队列、再非核心、最后拒绝”的流程执行任务，实现线程复用和流量控制。**

## 一、Java 线程池的核心原理

> **Java 线程池通过复用固定数量的线程来执行大量任务，避免频繁创建和销毁线程带来的性能开销，并通过队列和拒绝策略实现流量控制。**

核心实现类：
 **`ThreadPoolExecutor`**

------

## 二、线程池的七大核心参数（必背 ⭐）

```
public ThreadPoolExecutor(
    int corePoolSize,          // 核心线程数
    int maximumPoolSize,       // 最大线程数
    long keepAliveTime,        // 非核心线程空闲存活时间
    TimeUnit unit,
    BlockingQueue<Runnable> workQueue, // 任务队列
    ThreadFactory threadFactory,        // 线程工厂
    RejectedExecutionHandler handler    // 拒绝策略
)
```

------

## 三、线程池执行流程（重点）

> **这是面试官最关心的部分**

### 任务提交顺序

```
1. 当前线程数 < corePoolSize
   → 创建核心线程执行任务

2. 线程数 ≥ corePoolSize
   → 任务进入 workQueue 等待

3. 队列满 且 线程数 < maximumPoolSize
   → 创建非核心线程执行任务

4. 队列满 且 线程数 = maximumPoolSize
   → 触发拒绝策略
```

📌 **一句话记忆**：

> **先核心 → 再队列 → 再非核心 → 最后拒绝**

------

## 四、线程回收机制

- 核心线程：

  - 默认不会回收

  - 可通过：

    ```
    allowCoreThreadTimeOut(true);
    ```

- 非核心线程：

  - 空闲超过 `keepAliveTime` 后回收

------

## 五、常见任务队列

| 队列                    | 特点     | 适用场景         |
| ----------------------- | -------- | ---------------- |
| `LinkedBlockingQueue`   | 无界     | 吞吐优先         |
| `ArrayBlockingQueue`    | 有界     | 流量控制         |
| `SynchronousQueue`      | 不存任务 | 高并发、快速响应 |
| `PriorityBlockingQueue` | 有优先级 | 任务分级         |

------

## 六、拒绝策略（必考）

| 策略                  | 行为           |
| --------------------- | -------------- |
| `AbortPolicy`         | 抛异常（默认） |
| `CallerRunsPolicy`    | 调用者线程执行 |
| `DiscardPolicy`       | 直接丢弃       |
| `DiscardOldestPolicy` | 丢弃最旧任务   |

------

## 七、线程池为什么高效？

1. 线程复用，减少创建/销毁成本
2. 控制并发度，防止资源耗尽
3. 提供任务缓冲和削峰能力
4. 支持监控和动态调整参数

------

## 九、面试官常见追问

- ❓ 为什么不推荐 Executors？
  - 无界队列 / 无界线程风险 OOM
- ❓ CPU 密集型线程池怎么设？
  - `CPU 核数 + 1`
- ❓ IO 密集型呢？
  - `2 × CPU 核数`