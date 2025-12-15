## Java 中如何创建多线程？

这是 **Java 基础多线程面试题**，核心是理解 **线程创建方式 + 优缺点 + 使用场景**。我按 **方式 → 示例 → 面试总结** 给你整理清晰版。

------

## 面试总结一句话

> **Java 创建多线程主要有四种方式：继承 Thread、实现 Runnable、实现 Callable + FutureTask，以及使用线程池，其中线程池是生产环境推荐方式。**

## 一、Java 创建多线程的方式

### 1️⃣ 继承 `Thread` 类

- **做法**：自定义类继承 `Thread`，重写 `run()` 方法，然后调用 `start()` 启动线程。

```
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread running: " + Thread.currentThread().getName());
    }
}

public class Test {
    public static void main(String[] args) {
        Thread t1 = new MyThread();
        t1.start();
    }
}
```

**特点**

- 简单，直观
- **缺点**：Java 单继承，不能再继承其他类

------

### 2️⃣ 实现 `Runnable` 接口

- **做法**：实现 `Runnable` 接口，重写 `run()` 方法，把对象交给 `Thread` 构造器启动。

```
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable running: " + Thread.currentThread().getName());
    }
}

public class Test {
    public static void main(String[] args) {
        Thread t = new Thread(new MyRunnable());
        t.start();
    }
}
```

**特点**

- 可以避免单继承限制
- 可被多个线程共享同一个 Runnable 实例（共享状态）

------

### 3️⃣ 实现 `Callable` 接口 + `FutureTask`

- **做法**：`Callable` 有返回值，`FutureTask` 用于封装 Callable 并获取结果

```
import java.util.concurrent.*;

class MyCallable implements Callable<Integer> {
    @Override
    public Integer call() {
        return 42;
    }
}

public class Test {
    public static void main(String[] args) throws Exception {
        Callable<Integer> callable = new MyCallable();
        FutureTask<Integer> futureTask = new FutureTask<>(callable);
        Thread t = new Thread(futureTask);
        t.start();
        System.out.println("Result: " + futureTask.get());
    }
}
```

**特点**

- 支持返回值和异常处理
- 可与线程池结合使用

------

### 4️⃣ 使用线程池（推荐生产环境）

- **做法**：通过 `Executors` 或 `ThreadPoolExecutor` 创建线程池

```
import java.util.concurrent.*;

public class Test {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(3);
        executor.submit(() -> System.out.println("ThreadPool running: " + Thread.currentThread().getName()));
        executor.shutdown();
    }
}
```

**特点**

- 重用线程，减少频繁创建销毁开销
- 便于控制线程数量和任务队列

------

### 5️⃣ Java 8 Lambda 简化写法

```
new Thread(() -> System.out.println("Lambda thread running")).start();
```

- 简化 Runnable 写法
- 可结合线程池更简洁

------

## 二、面试总结对比表

| 方式               | 是否有返回值 | 是否可复用 | 适用场景             |
| ------------------ | ------------ | ---------- | -------------------- |
| Thread             | ❌            | ❌          | 简单场景             |
| Runnable           | ❌            | ✅          | 共享状态、复用线程   |
| Callable+Future    | ✅            | ✅          | 需要返回值或异常处理 |
| ThreadPoolExecutor | ✅            | ✅          | 高并发生产环境       |

------

