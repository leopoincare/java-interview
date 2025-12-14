Java 中 HashMap 的原理？
在 Java 中，HashMap 是最常用的集合之一，本质是一个基于哈希表的 key-value 容器。下面从结构、存取流程、冲突处理、扩容机制、JDK1.8 优化点几个方面系统说明它的原理。
一、整体结构
1️⃣ 底层数据结构
JDK 1.7 及以前：
👉 数组 + 链表
JDK 1.8 及以后：
👉 数组 + 链表 + 红黑树
table（数组）
 ├─ Node(key, value, hash, next)  → 链表
 ├─ TreeNode(...)                 → 红黑树
数组中的每个元素称为一个 桶（bucket）。

二、HashMap 的核心属性
static final int DEFAULT_INITIAL_CAPACITY = 16;
static final float DEFAULT_LOAD_FACTOR = 0.75f;
static final int TREEIFY_THRESHOLD = 8;
static final int UNTREEIFY_THRESHOLD = 6;
参数	含义
初始容量	默认 16（2 的幂）
负载因子	默认 0.75
树化阈值	链表长度 ≥ 8
退化阈值	树节点 ≤ 6
三、put() 存储原理
1️⃣ 计算 hash
hash = (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
🔹 高位参与运算，减少 hash 冲突
🔹 null key 的 hash 值固定为 0
2️⃣ 定位桶位置
index = (n - 1) & hash
⚠️ 数组长度必须是 2 的幂，才能用位运算代替取模，提高性能
3️⃣ 存储流程
桶为空 → 直接插入
桶不为空：
key 相同 → 覆盖 value
key 不同：
链表长度 < 8 → 尾插法
链表长度 ≥ 8 → 转红黑树（treeify）
四、get() 查询原理
根据 key 计算 hash
定位桶 index
判断：
桶头 key 相同 → 直接返回
链表 → 遍历比较
红黑树 → O(log n) 查找
五、hash 冲突解决方式
冲突原因
不同 key 计算出的 index 相同
解决方案
情况	方式
少量冲突	链表
大量冲突	红黑树
👉 JDK 1.8 引入红黑树，最坏时间复杂度从 O(n) → O(log n)
六、扩容（resize）机制 ⭐重点
1️⃣ 触发条件
size > capacity * loadFactor
默认：
16 * 0.75 = 12
2️⃣ 扩容过程
容量 × 2
节点重新分布
JDK 1.8 的优化点（面试高频）
不再重新计算 hash，而是：
(hash & oldCap) == 0
为 0 → 位置不变
不为 0 → index + oldCap
🚀 避免 rehash，效率更高
七、为什么 HashMap 线程不安全？
put、resize 无锁
并发扩容可能导致：
数据覆盖
丢失
JDK1.7 中甚至会形成 死循环
多线程场景应使用：
ConcurrentHashMap
Collections.synchronizedMap
八、HashMap 的几个经典面试点
1️⃣ 为什么容量是 2 的幂？
位运算更快
hash 分布更均匀
减少冲突
2️⃣ 为什么默认负载因子是 0.75？
空间利用率 vs 冲突概率 的折中
实践证明综合性能最优
3️⃣ 为什么树化阈值是 8？
泊松分布下，链表长度 ≥ 8 概率极低
防止过早树化增加开销
九、一句话总结（面试）
HashMap 基于数组实现，通过 hash 定位桶位置，采用链表和红黑树解决冲突，当元素超过阈值时进行扩容，JDK 1.8 通过红黑树和高效扩容策略显著提升了性能。
