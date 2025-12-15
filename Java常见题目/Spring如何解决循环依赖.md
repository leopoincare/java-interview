## Spring如何解决循环依赖的

Spring **解决循环依赖**的核心是通过 **三级缓存 + Bean 生命周期管理 + 延迟暴露引用** 来实现的。下面我系统讲清楚。

------

## 面试总结一句话

> **Spring 通过三级缓存 + Bean 生命周期管理 + 延迟暴露 Bean 引用的机制，解决了单例 Bean 的循环依赖问题，同时保证 AOP 代理对象注入一致性。**

## 一、循环依赖的定义

> **循环依赖**指的是两个或多个 Bean 互相依赖，形成闭环。例如：A 依赖 B，B 依赖 A。

**示例：**

```
@Component
class A {
    @Autowired
    private B b;
}

@Component
class B {
    @Autowired
    private A a;
}
```

------

## 二、Spring 解决循环依赖的前提

1. **只支持单例 Bean**
2. **必须是 setter 注入或字段注入**
   - 构造器注入无法解决
3. **Bean 生命周期由 Spring 托管**

------

## 三、Spring 循环依赖的三级缓存机制

Spring 在 `DefaultSingletonBeanRegistry` 中使用 **三级缓存**：

| 缓存     | 名称                    | 内容                                        |
| -------- | ----------------------- | ------------------------------------------- |
| 一级缓存 | `singletonObjects`      | 完整初始化好的 Bean                         |
| 二级缓存 | `earlySingletonObjects` | 半成品 Bean（未完成初始化，但可以依赖注入） |
| 三级缓存 | `singletonFactories`    | ObjectFactory，用于生成代理对象（如 AOP）   |

------

## 四、三级缓存的流程

假设 A <-> B 循环依赖，并且 A 可能被代理（AOP）：

1. **创建 A 实例**

   - A 实例化完成（constructor 执行完），但还未注入 B

2. **将 ObjectFactory 放入三级缓存**

   ```
   singletonFactories.put("A", () -> getEarlyBeanReference(A))
   ```

3. **创建 B 实例**

   - B 依赖 A → 从三级缓存调用 ObjectFactory 获取 A
   - 生成代理对象（如果需要 AOP）
   - 将 A 的代理对象放入二级缓存

4. **B 注入 A 的代理对象**

5. **A 完成依赖注入** → 放入一级缓存

✅ **整个过程中，容器中只会存在一个 A 的形态（代理对象）**

------

## 五、为什么三级缓存必不可少

- **二级缓存**只暴露半成品 Bean，无法区分“原始对象”和“代理对象”
- **三级缓存**通过 ObjectFactory 延迟生成代理对象，保证循环依赖中注入的 Bean 与最终使用 Bean 一致

------

## 六、关键点总结（面试重点）

1. **解决单例循环依赖**
2. **setter 注入可解决，构造器注入不行**
3. **三级缓存的作用**
   - 一级缓存：完整 Bean
   - 二级缓存：半成品 Bean
   - 三级缓存：延迟生成代理对象
4. **AOP Bean 可正确注入**

------

