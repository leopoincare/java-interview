## 线程的生命周期在 Java 中是如何定义的？

在 **Java 中，线程的生命周期**是 JVM 定义的标准模型，也是多线程面试高频题。下面我帮你整理成 **生命周期状态 → 转换条件 → 面试总结** 的清晰版。

------

## 面试总结一句话 

> **Java 线程生命周期包括 NEW → RUNNABLE → RUNNING → (BLOCKED / WAITING / TIMED_WAITING) → TERMINATED，不同状态通过 start、CPU 调度、锁等待、sleep/wait/join 等方法转换。**

------

## 一、Java 线程生命周期状态（Thread.State 枚举）

Java 中线程生命周期主要分为 **6 个状态**：

| 状态                          | 说明                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| **NEW（新建）**               | 线程对象被创建，但未调用 `start()` 方法                      |
| **RUNNABLE（可运行）**        | 线程已调用 `start()`，等待 CPU 调度执行                      |
| **BLOCKED（阻塞）**           | 线程等待获取锁（同步块/方法）                                |
| **WAITING（无限等待）**       | 线程调用 `Object.wait()`、`join()` 或 `LockSupport.park()`，无限期等待通知 |
| **TIMED_WAITING（限时等待）** | 线程调用带超时的方法，如 `sleep(ms)`、`wait(ms)`、`join(ms)` |
| **TERMINATED（终止）**        | 线程执行完 `run()` 方法，生命周期结束                        |

------

## 二、线程状态转换（核心）

```
NEW --start()--> RUNNABLE --CPU调度--> RUNNING
RUNNING --yield()--> RUNNABLE
RUNNING --sleep/wait(timeout)--> TIMED_WAITING
RUNNING --wait()--> WAITING
RUNNING --synchronized锁/阻塞队列--> BLOCKED
RUNNING --run()执行完成--> TERMINATED
```

### 转换说明

1. **NEW → RUNNABLE**
   - 调用 `start()`
   - 线程被提交给 JVM 调度
2. **RUNNABLE → RUNNING**
   - CPU 调度
   - 实际线程开始执行
3. **RUNNING → BLOCKED**
   - 遇到同步锁无法获得时阻塞
4. **RUNNING → WAITING / TIMED_WAITING**
   - 调用 `wait()` / `join()` → WAITING
   - 调用 `sleep()` / `wait(ms)` → TIMED_WAITING
5. **RUNNING → TERMINATED**
   - `run()` 方法执行完毕

------

## 三、状态图示（简化版）

```
         +-------+
         |  NEW  |
         +-------+
             |
         start()
             v
       +-----------+
       | RUNNABLE  | <-----------------------+
       +-----------+                         |
           | CPU调度                           |
           v                                   |
        +-------+                              |
        |RUNNING|                              |
        +-------+                              |
       /   |    \                              |
sleep/wait/join  synchronized(lock)           |
       |         |                             |
       v         v                             |
TIMED_WAITING   BLOCKED                        |
       |         |                             |
       +----notify/unlock---------------------+
       |
       v
   TERMINATED
```

