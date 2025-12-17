## Spring 启动过程？

 **Spring（以 Spring Boot 为例）的启动过程**，从高层到关键细节，分阶段说明：

------

## 面试总结一句话

> “Spring Boot 启动流程核心是：创建 SpringApplication → 准备 Environment → 创建 ApplicationContext → 加载 Bean 定义 → 实例化 Bean → 启动 Web 容器 → 发布启动事件。”

## 1️⃣ 启动入口

在 Spring Boot 应用中，通常有一个 `main` 方法：

```
public static void main(String[] args) {
    SpringApplication.run(MyApplication.class, args);
}
```

- **`SpringApplication.run`** 是整个启动过程的入口。
- 该方法会创建一个 `SpringApplication` 实例，然后调用 `run` 方法启动 Spring 上下文。

------

## 2️⃣ 初始化 `SpringApplication`

`SpringApplication` 构造流程主要做了几件事：

1. **判断应用类型**（Web/非Web）。
2. **准备 `ApplicationContext` 类型**：
   - `AnnotationConfigApplicationContext`（非Web）
   - `AnnotationConfigServletWebServerApplicationContext`（Web）
3. **准备启动监听器** `ApplicationListener`。
4. **设置默认属性**、环境变量 `Environment`、配置文件 `application.properties/yml`。

> 此阶段主要是做配置和准备，不会触发 Bean 实例化。

------

## 3️⃣ 准备 `Environment` 和配置属性

- 创建 `ConfigurableEnvironment`（`StandardEnvironment` 或 `StandardServletEnvironment`）。
- 读取配置来源：
  - 系统环境变量
  - JVM 参数
  - `application.properties` 或 `application.yml`
  - 命令行参数
- 处理 `@PropertySource` 或 Spring Boot 自动配置的 `ConfigFileApplicationListener`。

------

## 4️⃣ 创建 `ApplicationContext`

核心逻辑在：

```
ApplicationContext context = createApplicationContext();
```

- 对于 **Spring Boot Web 应用**，会创建 `AnnotationConfigServletWebServerApplicationContext`。
- 对于 **普通应用**，会创建 `AnnotationConfigApplicationContext`。
- 初始化上下文主要准备 BeanFactory。

------

## 5️⃣ 绑定 `Environment` 和 `ApplicationContext` 事件

- 将 `Environment` 注入 `ApplicationContext`。
- 注册 **`ApplicationListener`**，用于监听上下文事件（如 `ApplicationStartedEvent`、`ApplicationReadyEvent` 等）。
- 添加 **`ApplicationContextInitializer`**（用于自定义上下文初始化逻辑）。

------

## 6️⃣ 执行 `postProcessApplicationContext`

- 调用 `SpringApplicationRunListener` 的 `contextPrepared`。
- 此阶段可以修改上下文 Bean 定义（`BeanFactoryPostProcessor`）或环境配置。

------

## 7️⃣ 加载 Bean 定义（核心点）

Spring Boot 的核心：

1. **扫描注解**：
   - `@Configuration`、`@ComponentScan`、`@EnableAutoConfiguration`。
2. **Spring Boot 自动配置**：
   - 通过 `@EnableAutoConfiguration` + `spring.factories` 找到自动配置类。
   - 根据条件注解 `@ConditionalOnClass`、`@ConditionalOnMissingBean` 等判断是否生效。
3. **注册所有 Bean 定义** 到 `BeanFactory`。

> 注意：此阶段还没有实例化单例 Bean。

------

## 8️⃣ 实例化 Bean（Spring 容器刷新）

`context.refresh()` 会触发：

1. **BeanFactoryPostProcessor** 执行：
   - `ConfigurationClassPostProcessor` 解析配置类。
2. **BeanDefinition 注册** 完毕后：
   - 创建单例 Bean（`@Component`、`@Service`、`@Controller`、`@Configuration`）。
   - 处理依赖注入（`@Autowired`、`@Value`）。
3. **AOP & 生命周期处理**：
   - 初始化 `BeanPostProcessor`（包括 AOP 代理）。
   - 执行 `InitializingBean.afterPropertiesSet()` 和 `@PostConstruct`。

------

## 9️⃣ 启动 Web 容器（仅 Web 应用）

- 对于 Web 应用，Spring Boot 内置 **Tomcat/Jetty/Undertow**：
  - `ServletWebServerFactory` 创建 Web 服务器。
  - 注册 DispatcherServlet、Filter、Listener。
- Web 服务启动后，Spring MVC 才可接收请求。

------

## 🔟 发布事件 & 回调

Spring Boot 启动完成后：

1. 发布事件：

   - `ApplicationStartedEvent`
   - `ApplicationReadyEvent`（容器已就绪）

2. 执行 **CommandLineRunner** 和 **ApplicationRunner** 回调：

   ```
   @Component
   public class MyRunner implements CommandLineRunner {
       @Override
       public void run(String... args) {
           System.out.println("应用启动完成");
       }
   }
   ```

------

## 总结流程图（简化版）

```
main()
  └─ SpringApplication.run()
       ├─ 初始化 SpringApplication
       ├─ 准备 Environment
       ├─ 创建 ApplicationContext
       ├─ 执行 ApplicationContextInitializer
       ├─ 加载 Bean 定义（自动配置 + 注解扫描）
       ├─ refresh() → 实例化 Bean、注入依赖
       ├─ 启动 Web 容器（如果是 Web 应用）
       └─ 发布事件 + 执行 CommandLineRunner
```

------

如果你愿意，我可以画一张 **完整的 Spring Boot 启动流程图**，把 `SpringApplication`、`ApplicationContext`、`Bean 加载`、`事件` 等都标出来，直观又清楚。

![img](https://i-blog.csdnimg.cn/img_convert/101803c0eaa32fdbd30cf90455c6c1f6.png)

![img](https://i-blog.csdnimg.cn/img_convert/fc9a004695ecfc839f22f774af266c6d.png)