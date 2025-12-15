## Java 中 volatile 关键字的作用是什么？

这是 **Java 并发面试的必考题**，`volatile` 看似简单，其实考的是 **JMM（Java 内存模型）+ 可见性 + 有序性**。我按 **作用 → 原理 → 不能做什么 → 使用场景 → 面试总结** 给你讲清楚。

------

## 面试总结一句话

> **volatile 用于保证变量的可见性和有序性，但不保证原子性，适合状态标志和轻量级同步场景，底层通过内存屏障实现。**

------

## 一、volatile 的作用（核心三点 ⭐）

### 1️⃣ 保证**可见性**

- 一个线程修改了 `volatile` 变量
- **其他线程立刻能看到最新值**
- 不会从线程工作内存中读旧值

```
volatile boolean flag = true;

while (flag) {
    // do something
}
// 另一个线程 flag = false; 能立即跳出循环
```

------

### 2️⃣ 保证**有序性**（禁止指令重排序）

- `volatile` 写：
  - 前面的指令不能重排到后面
- `volatile` 读：
  - 后面的指令不能重排到前面

📌 通过 **内存屏障（Memory Barrier）** 实现

------

### 3️⃣ 不保证**原子性**（重点易错）

```
volatile int count = 0;

count++; // 非原子操作
```

- `count++` 实际是三步：
  - 读 → 加一 → 写
- 多线程下仍然会出现线程安全问题

------

## 二、volatile 的底层原理（面试加分）

### 1️⃣ Java 内存模型（JMM）

- 每个线程有自己的 **工作内存**
- volatile 变量直接读写 **主内存**

### 2️⃣ 内存屏障

- volatile 写 → Store Barrier
- volatile 读 → Load Barrier

👉 保证 **可见性 + 有序性**

------

## 三、volatile vs synchronized

| 对比   | volatile | synchronized |
| ------ | -------- | ------------ |
| 可见性 | ✅        | ✅            |
| 原子性 | ❌        | ✅            |
| 有序性 | ✅        | ✅            |
| 阻塞   | ❌        | ✅            |
| 性能   | 高       | 相对低       |

------

## 四、volatile 的典型使用场景

### 1️⃣ 状态标志位

```
volatile boolean running = true;
```

### 2️⃣ 双重检查锁（DCL）

```
private static volatile Singleton instance;

public static Singleton getInstance() {
    if (instance == null) {
        synchronized (Singleton.class) {
            if (instance == null) {
                instance = new Singleton();
            }
        }
    }
    return instance;
}
```

👉 防止指令重排导致“半初始化对象”

------

## 五、什么时候不能用 volatile？

- 需要 **复合操作原子性**
  - 如 `i++`、`check-then-act`
- 需要线程互斥访问
  - 临界区逻辑

👉 用 `synchronized` / `Lock` / `Atomic`

------

## 七、面试官常见追问（提前准备）

- ❓ volatile 为什么不能保证原子性？
- ❓ volatile 如何防止指令重排？
- ❓ synchronized 底层是否用 volatile？
  - 是的，用于锁状态标记