## Java的AQS

这是 **Java 并发面试经典题**，AQS（AbstractQueuedSynchronizer）是 **Java 并发包（java.util.concurrent）核心组件之一**，很多锁、同步器都是基于它实现的。下面我按 **概念 → 核心机制 → 应用 → 面试总结** 给你整理清晰版。

------

## 面试总结一句话

> **AQS 是 Java 并发包中用于构建锁和同步器的基础框架，核心是通过 state 状态和 FIFO 队列管理线程等待与唤醒，实现独占锁与共享锁的统一机制。**

------

## 一、AQS 的概念

> **AQS 是 Java 提供的一个用于构建锁和同步器的框架，基于 FIFO 队列实现线程等待与唤醒机制。**

### 核心作用

- 管理线程获取同步状态的队列
- 实现**可重入锁（ReentrantLock）、读写锁（ReentrantReadWriteLock）、Semaphore、CountDownLatch**等

------

## 二、AQS 核心机制

### 1️⃣ 内部状态（state）

- AQS 用一个 `int state` 变量表示同步器状态
- 由子类通过：
  - `getState()`
  - `setState(int)`
  - `compareAndSetState(int, int)` 来操作

### 2️⃣ 队列（CLH 队列）

- FIFO 队列管理等待线程
- **节点结构 Node**：
  - 包含线程引用、前驱/后继节点、等待状态（SIGNAL、CANCELLED、CONDITION 等）

### 3️⃣ 独占锁与共享锁

- **独占模式（Exclusive）**
  - 只有一个线程能获取锁（如 ReentrantLock）
- **共享模式（Shared）**
  - 多个线程可以同时获取（如 Semaphore、CountDownLatch）

### 4️⃣ 获取与释放流程（以独占锁为例）

```
1. 尝试获取锁
2. 获取失败 -> 将线程加入 CLH 队列尾部
3. 队列前驱节点释放锁 -> 唤醒后继线程
4. 重复尝试获取锁
```

------

## 三、AQS 的应用示例

### 1️⃣ ReentrantLock

```
Lock lock = new ReentrantLock();
lock.lock();
try {
    // 临界区
} finally {
    lock.unlock();
}
```

- 内部通过 AQS 的 **独占模式** 实现

### 2️⃣ CountDownLatch

```
CountDownLatch latch = new CountDownLatch(3);
latch.await(); // 阻塞等待
latch.countDown(); // 每调用一次减少计数
```

- 内部通过 **共享模式** 实现

------

## 四、AQS 的优势

1. **统一框架**：不用每个锁单独管理队列
2. **高性能**：使用 CAS + 队列 + 阻塞机制
3. **可扩展**：支持独占锁、共享锁、信号量等

------

## 面试可能追问

- ❓ AQS 中队列为什么用 CLH 队列？
  - ✅ 避免唤醒时线程争用，FIFO 保证公平性
- ❓ ReentrantLock 是独占锁还是共享锁？
  - ✅ 独占锁
- ❓ Condition 是如何实现等待/通知的？
  - ✅ 通过 AQS 内部的 ConditionObject，线程挂在等待队列，await/signal 唤醒