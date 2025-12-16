## Synchronized 和 ReentrantLock 有什么区别

这是 **Java 并发面试常考题**，面试官考察你对 **JVM 内置锁 vs 显式锁** 的理解。下面按 **概念 → 特性对比 → 典型场景 → 面试总结** 梳理清楚。

## 面试总结一句话

> **synchronized 是 JVM 内置锁，自动加锁解锁，适合简单场景；ReentrantLock 是显式锁，支持可中断、公平锁、尝试加锁和多条件变量，适合复杂并发控制。**

------

## 一、概念

| 名称            | 类型       | 描述                                                         |
| --------------- | ---------- | ------------------------------------------------------------ |
| `synchronized`  | JVM 内置锁 | 修饰方法或代码块，自动加锁解锁，阻塞式                       |
| `ReentrantLock` | 显式锁     | `java.util.concurrent.locks` 提供，需手动加锁解锁，支持高级功能 |

------

## 二、主要区别对比

| 对比项     | synchronized          | ReentrantLock                               |
| ---------- | --------------------- | ------------------------------------------- |
| 加锁方式   | 内置关键字            | API 显式调用 `lock()` / `unlock()`          |
| 是否可重入 | ✅                     | ✅                                           |
| 可中断     | ❌                     | ✅（`lockInterruptibly()`）                  |
| 可尝试加锁 | ❌                     | ✅（`tryLock()`）                            |
| 公平锁     | ❌（JVM 实现不可配置） | ✅（构造函数可选公平锁）                     |
| 性能       | JVM 优化后性能好      | Java 层实现，早期性能略低，但支持自定义策略 |
| 条件变量   | ❌                     | ✅（`Condition` 可实现多条件等待/通知）      |
| 使用复杂度 | 简单，自动释放        | 需手动释放，易忘记导致死锁                  |

------

## 三、使用场景

### 1️⃣ synchronized

- **方法或对象锁保护简单临界区**
- 对象粒度小，锁简单，不要求公平性
- 推荐用于 **短时间锁** 或 **代码块**

### 2️⃣ ReentrantLock

- 需要 **尝试加锁 / 可中断 / 公平锁**
- 高并发、复杂条件同步
- **Condition** 实现多线程等待/通知

------

## 四、底层实现区别（面试加分）

- `synchronized`
  - **JVM 内置**，依赖对象头 Mark Word + Monitor
  - 支持 **偏向锁、轻量级锁、重量级锁**
  - lock/unlock 自动由 JVM 处理
- `ReentrantLock`
  - **Java 层实现**
  - 内部通过 `AbstractQueuedSynchronizer (AQS)` 实现队列阻塞机制
  - 可选择公平或非公平策略

------

## 五、示意代码

```
// synchronized
synchronized(obj) {
    // 临界区
}

// ReentrantLock
Lock lock = new ReentrantLock();
lock.lock();
try {
    // 临界区
} finally {
    lock.unlock();
}
```

------

> 