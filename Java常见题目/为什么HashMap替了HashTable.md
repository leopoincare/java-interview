## 为什么HashMap替了HashTable

这是 **Java 集合面试的经典问题**，本质考察：**并发模型、性能、设计思想的演进**。我给你一套 **“原因 → 对比 → 一句话总结”** 的标准答案。

------

## 面试总结一句话（必背 ⭐）

> **Hashtable 因为方法级同步导致性能差、设计陈旧，被更高性能、更灵活的 HashMap 体系取代。**

------

## 30 秒压缩版（面试直接用）

> Hashtable 全方法加锁并发性能差，而且设计过时；HashMap 性能更高，并发场景用 ConcurrentHashMap，所以 Hashtable 被淘汰。

------

## 一、先给结论（一句话）

> **Hashtable 被 HashMap 取代，是因为 Hashtable 采用方法级同步，性能差、设计陈旧，而 HashMap 提供了更高性能和更灵活的并发方案。**

------

## 二、核心原因拆解（面试重点 ⭐）

### 1️⃣ 同步方式不同（最关键）

| 维度     | Hashtable             | HashMap |
| -------- | --------------------- | ------- |
| 线程安全 | 是                    | 否      |
| 同步方式 | 方法级 `synchronized` | 无同步  |
| 并发性能 | 极低                  | 高      |

- Hashtable：

```
public synchronized V get(Object key) { ... }
```

- **所有操作串行化**
- 多线程下 **严重阻塞**

👉 **HashMap 把“是否线程安全”交给使用者选择**

------

### 2️⃣ 性能差距巨大

- Hashtable：
  - 读也加锁
  - 写也加锁
  - 单线程性能尚可
  - 多线程性能崩
- HashMap：
  - 无锁
  - 单线程性能更好
  - 并发用 `ConcurrentHashMap`

------

### 3️⃣ 设计过时 & API 不友好

#### Hashtable 的“历史包袱”

- 属于 JDK 1.0
- 使用 `Enumeration`（非 Iterator）
- 方法名古老
- 扩展性差

HashMap：

- JDK 1.2 引入
- 属于 Java Collections Framework
- API 统一、设计现代

------

### 4️⃣ Null 值支持差异（细节追问）

| 集合      | null key | null value |
| --------- | -------- | ---------- |
| Hashtable | ❌        | ❌          |
| HashMap   | ✔        | ✔          |

📌 这是设计选择：

- Hashtable 用 null 表示失败
- HashMap 用异常区分

------

### 5️⃣ 并发场景有更好的替代品

- Hashtable ❌
- `Collections.synchronizedMap()` ⚠️（性能一般）
- **ConcurrentHashMap ✅（最佳实践）**

------

## 三、HashMap 不是线程安全，那为什么还要用？

> **单线程场景占绝大多数，HashMap 更快、更轻量；并发场景用 ConcurrentHashMap。**

------

## 四、面试官追问你可以这样接

- Q：那为什么不改造 Hashtable？

- A：

  > **因为设计层面已经落后，HashMap + ConcurrentHashMap 覆盖了全部场景，Hashtable 没有存在价值。

## **HashMap 进阶面试必问**：

- ❓ HashMap 为什么线程不安全？
- ❓ ConcurrentHashMap 是如何保证线程安全的？
- ❓ HashMap 在并发下会发生什么问题？
- ❓ Hashtable 有没有“安全”的使用场景？

## ❓ 1. HashMap 为什么线程不安全？

### 根本原因：**没有任何并发控制**

HashMap 的核心操作（put / resize / get）：

- **没有加锁**
- **没有 volatile**
- **没有 CAS**

👉 在多线程并发修改时，会破坏内部数据结构的一致性。

------

### 具体不安全点（面试官最爱）

#### ① put 过程非原子

- 计算 hash
- 定位桶
- 插入 / 覆盖
- 可能触发 resize

任一步被打断都会出问题。

------

#### ② resize 扩容过程危险（重点 ⭐）

- 多线程同时扩容
- 可能：
  - 数据丢失
  - 覆盖
  - 链表结构破坏

JDK 7 甚至可能出现 **死循环（CPU 100%）**

------

### 面试总结一句话 ⭐

> **HashMap 在并发场景下因为无锁保护，put 和 resize 过程会破坏数据结构，导致数据丢失甚至死循环。**

------

## ❓ 2. ConcurrentHashMap 是如何保证线程安全的？

### 不同 JDK 版本实现不同（面试加分）

------

### JDK 7：**分段锁（Segment）**

- HashMap 拆成多个 Segment
- 每个 Segment 是一个小 HashMap
- 锁粒度是 Segment

```
ConcurrentHashMap
 ├─ Segment[0] 🔒
 ├─ Segment[1] 🔒
```

📌 并发度 = Segment 数

------

### JDK 8：**CAS + synchronized（推荐 ⭐）**

#### put 流程简化：

1️⃣ 定位桶
 2️⃣ 桶为空 → **CAS 插入**
 3️⃣ 桶非空 → **synchronized 锁住链表/红黑树**
 4️⃣ 扩容使用协作式迁移

------

### 核心思想

> **锁粒度从“整表” → “桶级别”，并大量使用无锁 CAS。**

------

### 面试总结一句话 ⭐

> **ConcurrentHashMap 通过分段锁（JDK7）或 CAS + synchronized（JDK8）实现高并发下的线程安全。**

------

## ❓ 3. HashMap 在并发下会发生什么问题？

### 可能的问题清单（面试必答）

1️⃣ **数据丢失**

- put 覆盖
- resize 冲突

2️⃣ **死循环（JDK 7）**

- 扩容链表成环
- CPU 100%

3️⃣ **读到脏数据**

- get 看到中间状态

4️⃣ **结构损坏**

- 链表断裂
- size 不准

------

### 面试总结一句话 ⭐

> **并发使用 HashMap 可能导致数据丢失、死循环、脏读等严重问题，绝不能在并发场景使用。**

------

## ❓ 4. Hashtable 有没有“安全”的使用场景？

### 结论先行

> **几乎没有推荐使用场景，但“单线程 + 旧系统兼容”勉强算。**

------

### 可以接受的场景（非常少）

1️⃣ **历史遗留系统**

- 早期 JDK 1.0 代码
- 改动成本极高

2️⃣ **极低并发、极简单使用**

- 不追求性能
- 不可修改代码

------

### 不推荐原因（面试官期待）

- 方法级 synchronized
- 并发性能极差
- 已有更优替代方案

------

### 推荐替代方案

| 场景     | 推荐                        |
| -------- | --------------------------- |
| 单线程   | HashMap                     |
| 并发     | ConcurrentHashMap           |
| 简单同步 | Collections.synchronizedMap |

------

### 面试总结一句话 ⭐

> **Hashtable 理论上线程安全，但性能极差，实际开发中应使用 ConcurrentHashMap 替代。**

------

## ⭐ 终极 4 连问 30 秒版（面试直接用）

> HashMap 没有并发控制，put 和 resize 会破坏结构；ConcurrentHashMap 通过分段锁或 CAS + synchronized 保证线程安全；并发用 HashMap 会导致数据丢失甚至死循环；Hashtable 虽然安全但性能差，基本已被淘汰。