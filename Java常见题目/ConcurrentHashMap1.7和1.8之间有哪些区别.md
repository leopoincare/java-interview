##  Java 中 ConcurrentHashMap 1.7 和 1.8 之间有哪些区别？

这是 **ConcurrentHashMap** 的**经典高频面试题**。JDK **1.7 → 1.8** 的变化非常大，几乎是一次**重构级升级**。下面从**结构、并发机制、性能、扩容、API 行为**几个维度系统对比。

## 面试一句话总结

> **ConcurrentHashMap 在 JDK 1.8 中彻底移除了 Segment，采用 CAS + synchronized 的更细粒度锁设计，并引入红黑树和协同扩容机制，使并发性能、扩展性和稳定性都显著优于 1.7。**

------

## 一、整体结论先给出（面试版）

> **JDK 1.7 的 ConcurrentHashMap 基于 Segment 分段锁；
> JDK 1.8 去掉 Segment，采用 CAS + synchronized + 红黑树，锁粒度更细，并发性能更高。**

------

## 二、核心结构对比

| 版本        | 底层结构                             |
| ----------- | ------------------------------------ |
| **JDK 1.7** | Segment 数组 + HashEntry 数组 + 链表 |
| **JDK 1.8** | Node 数组 + 链表 / 红黑树            |

### 1️⃣ JDK 1.7 结构（分段锁）

```
ConcurrentHashMap
 └─ Segment[]（默认 16）
      └─ HashEntry[]（桶）
           └─ 链表
```

- **Segment 继承 ReentrantLock**
- 一个 Segment 管理一部分桶
- 锁的是 **整个 Segment**

🔴 问题：

- 锁粒度较粗
- 并发度受 Segment 数量限制

------

### 2️⃣ JDK 1.8 结构（取消 Segment）

```
table[]
 ├─ Node → 链表
 ├─ TreeNode → 红黑树
```

- **没有 Segment**
- 锁粒度降到 **桶 / 链表 / 树**
- 引入 **CAS + synchronized**

------

## 三、并发控制机制差异（重点）

### 🔹 JDK 1.7：Segment 分段锁

- `put()`：
  - 先定位 Segment
  - 对 Segment 加锁
  - 再操作桶
- 并发度 = Segment 数量（默认 16）

```
segment.lock();
try {
   // put
} finally {
   segment.unlock();
}
```

❌ 缺点：

- 多线程落在同一 Segment 仍会阻塞
- 扩容需要锁 Segment

------

### 🔹 JDK 1.8：CAS + synchronized

#### put() 流程（简化）

1. **CAS** 插入空桶
2. 桶非空 → `synchronized` 锁住 **桶头节点**
3. 链表 ≥ 8 → 红黑树

```
if (casTabAt(...)) {
   // 无锁插入
} else {
   synchronized (f) {
       // 链表 / 树操作
   }
}
```

✅ 优点：

- 锁粒度更细
- 低冲突场景几乎无锁
- 高并发下吞吐量更高

------

## 四、扩容（resize）机制对比 ⭐

### 🔹 JDK 1.7

- 每个 Segment **独立扩容**
- 扩容时锁住整个 Segment
- rehash 成本高

------

### 🔹 JDK 1.8（革命性改进）

- **多线程协同扩容**
- 使用 `ForwardingNode` 标记已迁移桶
- 线程“帮忙”迁移数据

```
旧表 ──迁移──▶ 新表
     ↑
 ForwardingNode
```

🚀 优点：

- 扩容不会阻塞所有线程
- 大表扩容性能大幅提升

------

## 五、get() 读取性能差异

| 版本    | get 是否加锁 |
| ------- | ------------ |
| JDK 1.7 | ❌ 不加锁     |
| JDK 1.8 | ❌ 不加锁     |

但实现不同：

- **1.7**：volatile + unsafe 保证可见性
- **1.8**：volatile + Node 不可变设计

------

## 六、红黑树支持（1.8 新增）

| 版本    | 冲突处理      |
| ------- | ------------- |
| JDK 1.7 | 仅链表        |
| JDK 1.8 | 链表 → 红黑树 |

- 树化阈值：**8**
- 查找复杂度：`O(n)` → `O(log n)`

------

## 七、size() 实现差异

### 🔹 JDK 1.7

- 遍历所有 Segment
- 统计 count
- **可能不准确**
- 多次重试保证近似一致

------

### 🔹 JDK 1.8

- 使用 `baseCount + CounterCell[]`
- 类似 LongAdder
- 高并发下统计更快、更准

------

## 八、API 行为变化（1.8 新增）

### JDK 1.8 新增方法

```
map.forEach()
map.computeIfAbsent()
map.merge()
map.reduce()
```

- 原子操作
- 更适合并发统计场景

------

## 九、关键差异一览表（面试速记）

| 对比点    | JDK 1.7         | JDK 1.8            |
| --------- | --------------- | ------------------ |
| 并发控制  | Segment 分段锁  | CAS + synchronized |
| 锁粒度    | Segment         | 桶 / 节点          |
| 扩容      | 单 Segment 扩容 | 多线程协作扩容     |
| 数据结构  | 链表            | 链表 + 红黑树      |
| 并发度    | 固定（默认 16） | 动态，理论更高     |
| size 统计 | 遍历 Segment    | LongAdder 思路     |
| 性能      | 中              | 高（显著提升）     |

