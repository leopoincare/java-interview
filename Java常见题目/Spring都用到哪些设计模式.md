## Spring 都用到哪些设计模式？

Spring 框架可以说是 **设计模式的集大成者**，从核心容器到 MVC、AOP，几乎用到了大部分经典设计模式。下面我给你整理一个**面试友好清单**，并附使用场景说明。

------

## 面试总结一句话

> **Spring 核心容器使用单例、工厂模式实现 Bean 管理；AOP 使用代理模式增强对象功能；事件系统使用观察者模式；模板类使用模板方法模式；策略、责任链、适配器等模式贯穿整个框架，实现了高内聚低耦合。**

## 一、创建型模式（Creational Pattern）

| 模式                      | Spring 中应用                                 | 场景                                       |
| ------------------------- | --------------------------------------------- | ------------------------------------------ |
| **单例模式**              | Spring 默认 Bean 单例（Singleton Scope）      | 控制容器中 Bean 实例唯一，减少内存消耗     |
| **工厂模式 / 抽象工厂**   | `BeanFactory`、`ApplicationContext`           | 创建 Bean 对象，通过配置文件或注解生成实例 |
| **建造者模式（Builder）** | `UriComponentsBuilder`、`RestTemplateBuilder` | 构建复杂对象，如 URL、请求对象             |
| **原型模式（Prototype）** | Bean Scope=prototype                          | 每次获取 Bean 都生成新实例                 |
| **依赖注入（DI）**        | 本质是工厂 + 反射机制                         | 通过容器管理对象依赖                       |

------

## 二、结构型模式（Structural Pattern）

| 模式                        | Spring 中应用                                                | 场景                                       |
| --------------------------- | ------------------------------------------------------------ | ------------------------------------------ |
| **代理模式（Proxy）**       | Spring AOP（JDK 动态代理 / CGLIB）                           | 增强目标对象，做事务、日志、权限等横切功能 |
| **装饰器模式（Decorator）** | `RestTemplate`、`JdbcTemplate`                               | 给对象动态添加功能，不改变原对象           |
| **适配器模式（Adapter）**   | `HandlerAdapter`、`JdbcTemplate`                             | 将不同接口统一到容器或调用方式             |
| **桥接模式（Bridge）**      | `Resource` 抽象接口 + `FileSystemResource` / `ClassPathResource` | 将抽象与实现解耦                           |
| **组合模式（Composite）**   | `ApplicationContext` 父子容器                                | 父容器和子容器统一管理 Bean，层级组织      |

------

## 三、行为型模式（Behavioral Pattern）

| 模式                                      | Spring 中应用                                                | 场景                                 |
| ----------------------------------------- | ------------------------------------------------------------ | ------------------------------------ |
| **观察者模式（Observer）**                | `ApplicationEventPublisher` + `@EventListener`               | 事件发布与订阅，如系统事件通知       |
| **模板方法模式（Template Method）**       | `JdbcTemplate`、`RestTemplate`、`AbstractApplicationContext` | 定义骨架逻辑，子类实现具体步骤       |
| **策略模式（Strategy）**                  | `PlatformTransactionManager`、`CacheManager`                 | 不同策略可替换，如事务策略、缓存策略 |
| **责任链模式（Chain of Responsibility）** | Spring MVC `HandlerInterceptor`、`FilterChain`               | 请求处理链条，按顺序处理请求         |
| **命令模式（Command）**                   | `TaskExecutor` + `Runnable`                                  | 封装请求为对象，方便异步或定时执行   |
| **状态模式（State）**                     | `AbstractPlatformTransactionManager`                         | 事务状态机：新建、活动、提交、回滚等 |
| **备忘录模式（Memento）**                 | `BeanWrapper` 属性快照                                       | Bean 属性变化前可保存原值            |

------

> 