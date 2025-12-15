## Spring / Spring Boot中Bean Scope是什么？

在 **Spring / Spring Boot** 中，**Bean Scope（作用域）** 用来定义 **Spring 容器管理的 Bean 的生命周期和可见性范围**。也就是 **一个 Bean 的实例在容器中是单例、原型还是其他类型**。下面我帮你梳理清楚：

------

## 面试总结一句话

> **Bean Scope 定义了 Spring 容器管理 Bean 的生命周期和可见性，常见有 singleton（默认单例）、prototype（每次新建）、request、session 等 Web 范围，选择合适 Scope 可控制 Bean 的创建和共享策略。**

## 一、常用 Bean Scope

| Scope           | 生命周期                             | 说明                               | Spring Boot 默认       |
| --------------- | ------------------------------------ | ---------------------------------- | ---------------------- |
| **singleton**   | 容器启动时创建，整个容器只有一个实例 | 默认作用域，所有依赖共享同一个实例 | ✅ 默认                 |
| **prototype**   | 每次从容器获取都会创建新实例         | 容器只负责创建，不管理销毁         | ❌ 不会自动调用销毁方法 |
| **request**     | 每次 HTTP 请求都会创建一个 Bean 实例 | 仅用于 Web 应用                    | ❌ Web 环境             |
| **session**     | 每个 HTTP Session 创建一个 Bean 实例 | 仅用于 Web 应用                    | ❌ Web 环境             |
| **application** | 在 ServletContext 范围内共享         | 仅用于 Web 应用                    | ❌ Web 环境             |
| **websocket**   | 每个 WebSocket 会话一个实例          | 仅用于 WebSocket                   | ❌ Web 环境             |

------

## 二、定义 Bean Scope

### 1️⃣ 注解方式（最常用）

```
@Component
@Scope("prototype")
public class MyBean {}
```

### 2️⃣ XML 配置方式（老版本）

```
<bean id="myBean" class="com.example.MyBean" scope="prototype"/>
```

------

## 三、注意点

1. **singleton**
   - 默认作用域
   - 在 Spring Boot 容器启动时创建
   - 生命周期由容器管理
2. **prototype**
   - 容器只创建，不管理销毁
   - 需要自己在销毁前清理资源
   - 注入 prototype Bean 到 singleton 时要注意：
     - 可以使用 `@Lookup` 或 `ObjectProvider` 获取新的实例
3. **Web 范围 Beans**
   - 需要在 Web 环境下才生效
   - 例如 `request`、`session`

