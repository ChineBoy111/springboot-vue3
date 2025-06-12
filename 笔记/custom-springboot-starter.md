
# 🚀 Spring Boot 自定义 Starter 教程笔记

本笔记介绍如何创建一个可复用的 Spring Boot 自定义 Starter 模块，实现自己的自动配置。

---

## ✅ 一、Starter 的基本概念

### 1. 什么是 Starter？

Spring Boot Starter 是一个用于简化依赖管理的模块，通常包含：

- 相关依赖包；
- 自动配置类；
- 与应用解耦的功能实现。

例如：`spring-boot-starter-web` 会自动引入 Web 开发所需的组件（Spring MVC、Tomcat等）。

---

## ✅ 二、自定义 Starter 的结构

一个完整的 Starter 通常包含两个模块：

```
my-spring-boot-starter/
├── my-spring-boot-starter-autoconfigure   # 自动配置模块
└── my-spring-boot-starter                 # Starter 引用模块
```

---

## ✅ 三、开发步骤详解

### 📁 1. 创建自动配置模块 `my-spring-boot-starter-autoconfigure`

#### 1.1 添加依赖（`pom.xml`）

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure</artifactId>
    </dependency>
</dependencies>
```

#### 1.2 编写自动配置类

```java
@Configuration
@ConditionalOnClass(MyService.class)
@EnableConfigurationProperties(MyProperties.class)
public class MyServiceAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public MyService myService(MyProperties properties) {
        return new MyService(properties.getPrefix());
    }
}
```

#### 1.3 编写属性类

```java
@ConfigurationProperties(prefix = "my.service")
public class MyProperties {
    private String prefix = "hello";

    public String getPrefix() { return prefix; }
    public void setPrefix(String prefix) { this.prefix = prefix; }
}
```

#### 1.4 注册自动配置类

在 `resources/META-INF/spring.factories`（Spring Boot 2.x） 或  
`resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`（Spring Boot 3.x） 中添加：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.autoconfigure.MyServiceAutoConfiguration
```

---

### 📁 2. 创建 Starter 模块 `my-spring-boot-starter`

该模块只需依赖自动配置模块：

```xml
<dependencies>
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>my-spring-boot-starter-autoconfigure</artifactId>
        <version>1.0.0</version>
    </dependency>
</dependencies>
```

---

## ✅ 四、使用自定义 Starter

在实际项目中引入自定义 Starter：

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>my-spring-boot-starter</artifactId>
    <version>1.0.0</version>
</dependency>
```

配置属性（可选）：

```yaml
my:
  service:
    prefix: "SpringBoot"
```

调用：

```java
@RestController
public class HelloController {
    @Autowired
    private MyService myService;

    @GetMapping("/hello")
    public String hello() {
        return myService.sayHello();
    }
}
```

---

## ✅ 五、常见注解说明

| 注解 | 说明 |
|------|------|
| `@EnableConfigurationProperties` | 启用配置属性类 |
| `@ConfigurationProperties` | 绑定配置文件中的值 |
| `@ConditionalOnClass` | classpath 中存在某类才生效 |
| `@ConditionalOnMissingBean` | 容器中没有此 Bean 才注入 |

---

## ✅ 六、自定义 Starter 优势

- 实现功能复用（多个项目统一使用）
- 与业务系统解耦，便于维护
- 灵活的配置与扩展机制

---

## ✅ 七、总结

1. Starter 本质是一个包含自动配置类的依赖模块；
2. 使用条件注解控制自动装配逻辑；
3. 提供默认配置，允许用户通过配置覆盖；
4. 可发布到私服或开源平台，实现模块共享。

---
