## SpringBoot 是如何实现自动配置的？

这是 **Spring Boot 面试必考题**。我按 **“原理 → 核心流程 → 关键机制 → 面试一句话”** 给你讲清楚。

------

## 面试总结一句话

> **Spring Boot 通过 `@EnableAutoConfiguration` 加载自动配置类，再结合条件注解和配置属性绑定，在容器启动时按需自动注册 Bean，实现开箱即用。**

## 一、什么是 Spring Boot 自动配置？

> **自动配置**：Spring Boot 根据 **当前类路径（classpath）**、**已有 Bean** 和 **配置文件**，自动帮我们配置合适的 Bean，做到“约定大于配置”。

------

## 二、自动配置的入口：`@SpringBootApplication`

```
@SpringBootApplication
public class Application {}
```

等价于：

```
@Configuration
@ComponentScan
@EnableAutoConfiguration
```

👉 **自动配置的核心是 `@EnableAutoConfiguration`**

------

## 三、自动配置的核心流程（重点）

### 1️⃣ `@EnableAutoConfiguration`

```
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}
```

- 通过 `@Import` 导入 **AutoConfigurationImportSelector**
- 该类负责 **加载所有自动配置类**

------

### 2️⃣ 从哪里加载自动配置类？

#### Spring Boot 2.x 及之前

- 从：

```
META-INF/spring.factories
```

- Key：

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration
```

#### Spring Boot 3.x

- 改为：

```
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

👉 文件中列出了 **所有自动配置类**（如 `DataSourceAutoConfiguration`）

------

### 3️⃣ 自动配置类是普通的 `@Configuration`

示例：

```
@Configuration
@ConditionalOnClass(DataSource.class)
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public DataSource dataSource() {
        ...
    }
}
```

------

## 四、自动配置为什么“智能”？

### 1️⃣ 条件注解（核心）

| 注解                           | 作用                     |
| ------------------------------ | ------------------------ |
| `@ConditionalOnClass`          | 类路径存在某个类才生效   |
| `@ConditionalOnMissingBean`    | 容器中没有该 Bean 才创建 |
| `@ConditionalOnProperty`       | 配置存在且满足条件       |
| `@ConditionalOnBean`           | 容器中存在某个 Bean      |
| `@ConditionalOnWebApplication` | Web 环境才生效           |

👉 **条件注解决定“配不配”**

------

### 2️⃣ 配置属性绑定

```
@ConfigurationProperties(prefix = "spring.datasource")
public class DataSourceProperties {}
```

- 将 `application.yml` 中配置自动绑定到属性类
- 实现 **配置驱动自动装配**

------

### 3️⃣ 自动配置是“可覆盖”的

- 用户自己定义 Bean：

```
@Bean
public DataSource myDataSource() {}
```

- 自动配置中的：

```
@ConditionalOnMissingBean
```

会失效

👉 **用户配置优先级更高**

------

## 五、自动配置执行时机

- 发生在 **Spring 容器启动阶段**
- **BeanDefinition 注册阶段**
- 在 `refresh()` 之前完成

------

## 六、如何关闭或排除自动配置？

```
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
```

或：

```
spring.autoconfigure.exclude=xxxAutoConfiguration
```

## 七、面试追问常见问题

- ❓ 为什么引入 starter 就能用？
  - starter 引入依赖 + 自动配置类生效
- ❓ 自动配置和 `@ComponentScan` 区别？
  - 前者是 **条件装配 Bean**
  - 后者是 **扫描并注册 Bean**
- ❓ SpringBoot 自动配置核心类？
  - `AutoConfigurationImportSelector`