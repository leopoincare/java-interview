## Java的AQS实现

这是 **Java 并发面试必考点**，CAS（Compare And Swap）是 **无锁并发的核心基础**。我按 **概念 → 工作原理 → 优缺点 → ABA 问题 → 面试总结** 给你讲清楚。

## 面试总结一句话

> **CAS 是一种基于 CPU 原子指令的无锁并发机制，通过比较并交换实现原子更新，性能高但存在 ABA 问题，Java 通过 Atomic 类和 AQS 广泛使用 CAS。**

------

## 一、什么是 CAS？

> **CAS（Compare And Swap）是一种无锁并发机制，通过比较内存中的值和期望值是否相等，来决定是否更新为新值。**

### 核心思想

```
if (内存值 == 期望值) {
    内存值 = 新值;
} else {
    操作失败;
}
```

- 这是一个 **原子操作**
- 不需要加锁，性能高
- 底层由 CPU 指令保证（如 x86 的 `cmpxchg`）

------

## 二、CAS 的工作原理（Java 视角）

### 1️⃣ Unsafe + CPU 原子指令

- Java 通过 `Unsafe.compareAndSwapXxx`
- JDK 9+ 通过 `VarHandle`
- 最终映射到 CPU 原子指令

### 2️⃣ 自旋重试

```
while (!compareAndSet(oldValue, newValue)) {
    oldValue = get();
}
```

- CAS 失败不会阻塞线程
- 通过 **自旋** 反复尝试

------

## 三、CAS 在 Java 中的应用

### 1️⃣ Atomic 类

```
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet();
```

### 2️⃣ AQS / ReentrantLock

- AQS 内部用 CAS 修改 `state`

### 3️⃣ ConcurrentHashMap

- JDK 8 使用 CAS + synchronized

------

## 四、CAS 的优缺点

### 优点

- 无锁，避免线程上下文切换
- 性能好，适合高并发
- 不会死锁

### 缺点

1. **自旋消耗 CPU**
2. **ABA 问题**
3. 只能保证一个变量的原子性

------

## 五、ABA 问题

### 什么是 ABA？

- 内存值从 A → B → A
- CAS 比较发现仍是 A，误以为未发生变化

### 解决方案

- **版本号**

```
AtomicStampedReference<Integer> ref;
```

- **时间戳**

```
AtomicMarkableReference
```

------

## 六、CAS vs synchronized

| 对比项     | CAS  | synchronized |
| ---------- | ---- | ------------ |
| 是否阻塞   | 否   | 是           |
| 上下文切换 | 无   | 有           |
| 死锁       | 不会 | 可能         |
| 使用复杂度 | 高   | 低           |

------

## 八、面试官常见追问（提前准备）

- ❓ CAS 一定比锁快吗？
   👉 不是，高冲突下自旋浪费 CPU
- ❓ CAS 如何解决 ABA？
   👉 版本号 / AtomicStampedReference
- ❓ synchronized 底层是否用 CAS？
   👉 偏向锁、轻量级锁阶段使用 CAS