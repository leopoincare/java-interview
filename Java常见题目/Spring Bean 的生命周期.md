## 说下 Spring Bean 的生命周期？

Spring Bean 的生命周期是一个核心概念，理解它对掌握 Spring 框架至关重要。下面是 Bean 从创建到销毁的完整生命周期，可以分为几个关键阶段：

## 面试总结一句话 

> **Spring Bean 生命周期包括实例化 → 属性注入 → Aware 回调 → 初始化前处理 → 初始化 → 初始化后处理 → 可用 → 容器销毁，BeanPostProcessor 和初始化/销毁回调是扩展关键点。**

------

**更精炼版本（按需选用）：**

- **技术向：** Spring Bean 生命周期由容器管理，核心是实例化、属性注入、初始化和销毁四个阶段，通过 Aware、BeanPostProcessor 等接口提供扩展点。
- **面试向：** Bean 生命周期就是 Spring 容器创建、初始化、使用和销毁 Bean 的完整过程，理解其阶段和扩展点是掌握 IoC 容器的关键。
- **对比向：** 相比普通对象的 new 和 GC，Spring Bean 多了依赖注入、容器回调、AOP 代理等中间步骤，体现了框架的管控能力。

这是 **Spring 面试经典题**，核心是把 Bean **创建、初始化、销毁** 全流程讲清楚。下面我按 **完整流程 → 关键扩展点 → 面试一句话** 来总结。

------

## 一、Spring Bean 生命周期概览

Spring Bean 的生命周期分为几个阶段：

1. **实例化（Instantiation）**
   - Spring 通过 **反射调用构造函数** 创建 Bean 对象
   - Bean 对象还没有注入属性
2. **属性赋值（Populate Properties）**
   - Spring 将配置的依赖注入（`@Autowired` / XML 配置 / setter）
   - Bean 对象已经有属性，但还未初始化
3. **BeanNameAware / BeanFactoryAware / ApplicationContextAware 回调**
   - Bean 可以获取自身名称、BeanFactory 或 ApplicationContext
   - 常用于获取环境信息或上下文对象
4. **BeanPostProcessor 前置处理（Before Initialization）**
   - 调用 `postProcessBeforeInitialization()`
   - 可以修改 Bean 对象或进行代理处理（如 AOP）
5. **初始化（Initialization）**
   - `@PostConstruct` 注解方法
   - 实现 `InitializingBean.afterPropertiesSet()` 方法
6. **BeanPostProcessor 后置处理（After Initialization）**
   - 调用 `postProcessAfterInitialization()`
   - 通常用于生成代理对象（AOP、动态代理）
7. **Bean 可用（Ready to Use）**
   - Bean 被注入到容器中，供业务使用
8. **销毁（Destruction）**
   - 容器关闭时调用
   - 通过 `@PreDestroy` 或实现 `DisposableBean.destroy()`
   - 可以释放资源

------

## 二、Bean 生命周期顺序图（简化版）

```
实例化 → 属性赋值 → Aware 回调 → postProcessBeforeInitialization → 初始化回调 → postProcessAfterInitialization → 可用 → 销毁
```

------

## 三、关键扩展点

| 扩展点                                    | 用途                |
| ----------------------------------------- | ------------------- |
| `BeanPostProcessor`                       | 修改 Bean、生成代理 |
| `InitializingBean` / `afterPropertiesSet` | 初始化逻辑          |
| `@PostConstruct`                          | 注解方式初始化      |
| `DisposableBean` / `destroy()`            | 销毁逻辑            |
| `@PreDestroy`                             | 注解方式销毁        |

## **四、 示例代码（演示关键阶段）**

java

```
@Component
public class MyBean implements BeanNameAware, InitializingBean, DisposableBean {
    
    @Value("${some.property}")
    private String property;
    
    // 1. 构造方法（实例化）
    public MyBean() {
        System.out.println("1. 实例化");
    }
    
    // 2. 属性注入（由 Spring 自动完成）
    
    // 3. Aware 接口
    @Override
    public void setBeanName(String name) {
        System.out.println("3. BeanNameAware: " + name);
    }
    
    // 4-5. 初始化
    @PostConstruct
    public void postConstruct() {
        System.out.println("5.1 @PostConstruct");
    }
    
    @Override
    public void afterPropertiesSet() {
        System.out.println("5.2 InitializingBean.afterPropertiesSet()");
    }
    
    public void customInit() {
        System.out.println("5.3 自定义 init-method");
    }
    
    // 6. 后置处理（由 BeanPostProcessor 执行，此处不展示）
    
    // 8-9. 销毁
    @PreDestroy
    public void preDestroy() {
        System.out.println("9.1 @PreDestroy");
    }
    
    @Override
    public void destroy() {
        System.out.println("9.2 DisposableBean.destroy()");
    }
}

// 定义 BeanPostProcessor
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        System.out.println("4. BeanPostProcessor.before: " + beanName);
        return bean;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("6. BeanPostProcessor.after: " + beanName);
        return bean;
    }
}
```



## **五、 不同作用域 Bean 的生命周期差异**

- **单例（Singleton）**：默认作用域，容器启动时创建（可配置懒加载），容器关闭时销毁。
- **原型（Prototype）**：每次获取时创建新实例，**容器不管理其完整生命周期**，初始化阶段会执行，但销毁方法不会被调用。
- **Web 作用域（Request、Session 等）**：生命周期与 Web 请求/会话绑定，创建和销毁由 Spring MVC 管理。

## **六、 总结记忆要点**

- 核心流程：**实例化 → 属性填充 → 初始化 → 销毁**
- 三个关键扩展点：**Aware 接口、BeanPostProcessor、生命周期回调**
- 两种注入时机：构造器注入在实例化时完成，Setter 注入在属性填充阶段
- 推荐使用注解：`@PostConstruct` 和 `@PreDestroy` 替代 Spring 接口

理解 Bean 的生命周期有助于解决依赖注入问题、实现自定义扩展（如 BeanPostProcessor）以及优化应用启动性能。