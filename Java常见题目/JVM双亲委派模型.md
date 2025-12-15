## jvm双亲委派模型

这是 **JVM 类加载机制面试常考题**，核心是理解 **为什么要双亲委派、工作流程、优缺点**。我按 **概念 → 流程 → 代码示意 → 面试总结** 来讲。

## 面试总结一句话

> **JVM 双亲委派模型是指类加载器在加载类时先委托父加载器，如果父加载器无法加载才由自己加载，以保证核心类安全、避免重复加载，并提高效率。**

------

## 一、概念

> **双亲委派模型（Parent Delegation Model）** 是 JVM 类加载器的一种设计模式，指类加载请求先委托给父类加载器，如果父加载器无法完成再由子加载器自己加载。

目的：

1. 避免重复加载同一个类
2. 保证核心类（`java.lang.*`）由 **启动类加载器** 加载，保证安全
3. 提高类加载效率

------

## 二、类加载器层次（JDK 8 为例）

| 类加载器                                         | 说明                              |
| ------------------------------------------------ | --------------------------------- |
| 启动类加载器（Bootstrap ClassLoader）            | 加载 JDK 核心类库（`rt.jar`）     |
| 扩展类加载器（Extension / Platform ClassLoader） | 加载 JDK 扩展库（`jre/lib/ext`）  |
| 应用类加载器（Application / System ClassLoader） | 加载应用类路径下的类（classpath） |
| 自定义类加载器                                   | 用户自定义，可继承 `ClassLoader`  |

------

## 三、双亲委派流程

假设 **应用类加载器**加载 `com.example.Foo`：

1. 应用类加载器接到加载请求
2. 委托给 **扩展类加载器**
3. 扩展类加载器接到请求 → 委托给 **启动类加载器**
4. 启动类加载器查找 `Foo`：
   - 找到 → 返回 Class
   - 未找到 → 返回 null，交给扩展类加载器自己加载
5. 扩展类加载器尝试加载
   - 找到 → 返回 Class
   - 未找到 → 返回 null，交给应用类加载器自己加载
6. 应用类加载器尝试加载 → 最终返回 Class 或抛异常

------

## 四、流程示意图

```
应用类加载器 (AppClassLoader)
        |
        v
扩展类加载器 (ExtClassLoader)
        |
        v
启动类加载器 (BootstrapClassLoader)
        |
        v
    查找核心类 java.lang.*
        |
        └─ 找到则返回
        └─ 找不到则由子加载器加载
```

------

## 五、代码示意（可在面试口述）

```
ClassLoader appLoader = ClassLoader.getSystemClassLoader();
System.out.println(appLoader); // AppClassLoader

ClassLoader extLoader = appLoader.getParent();
System.out.println(extLoader); // ExtClassLoader

ClassLoader bootstrapLoader = extLoader.getParent();
System.out.println(bootstrapLoader); // null (启动类加载器由C++实现)
```

------

## 六、优点

1. **避免类重复加载**
2. **保证核心类的安全性**
3. **提高加载效率**

------

## 七、缺点

- 有时需要 **破坏双亲委派加载**（如自定义类热部署、Tomcat WebApp）
   → 可通过 **自定义类加载器覆盖 `loadClass`** 实现

