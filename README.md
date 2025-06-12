# 记录学习Spingboot-vue3 NJUPT程序员课程的仓库

# Spring Boot 自动配置原理

## ✅ 一、自动配置的基本概念

Spring Boot 自动配置的核心目标是：

> **根据你引入的依赖和当前的配置，自动帮你配置 Spring 应用所需的 Bean。**

开发者无需手动编写繁杂的配置文件，Spring Boot 会根据条件自动装配 Bean，提供“开箱即用”的体验。

---

## ✅ 二、自动配置的核心机制

### 1. `@SpringBootApplication` 注解

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

`@SpringBootApplication` 是一个组合注解，包含：

- `@SpringBootConfiguration`：本质是 `@Configuration`
- `@EnableAutoConfiguration`：启用自动配置的关键
- `@ComponentScan`：组件扫描

---

### 2. `@EnableAutoConfiguration` 是关键

它通过 `@Import` 导入了一个类：

```java
@Import(AutoConfigurationImportSelector.class)
```

该类会读取自动配置类清单，并将其注册为 Bean。

---

### 3. 自动配置类的来源

Spring Boot 会从以下位置加载自动配置类：

```
META-INF/spring.factories （Spring Boot 2.x）
或
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports （Spring Boot 3.x）
```

示例内容：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
...
```

这些类本质是 `@Configuration` 配置类。

---

## ✅ 三、条件注解驱动自动配置

Spring Boot 使用大量条件注解决定是否执行某段配置：

| 注解 | 作用 |
|------|------|
| `@ConditionalOnClass` | classpath 中存在某类时生效 |
| `@ConditionalOnMissingBean` | 当前上下文中不存在指定 Bean 时生效 |
| `@ConditionalOnProperty` | 配置文件中存在某属性或值为指定值时生效 |
| `@ConditionalOnBean` | Bean 存在时生效 |

### 示例：`DispatcherServletAutoConfiguration`

```java
@Configuration
@ConditionalOnClass(Servlet.class)
@ConditionalOnWebApplication
public class DispatcherServletAutoConfiguration {
    @Bean
    public DispatcherServlet dispatcherServlet() {
        return new DispatcherServlet();
    }
}
```

---

## ✅ 四、自定义或覆盖自动配置

Spring Boot 支持用户通过以下方式自定义或覆盖默认行为：

### 1. 定义同名 Bean

只要你定义了自己的 Bean（如 `DataSource`），Spring Boot 就不会自动创建它。

### 2. 配置文件控制

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: 123456
```

自动配置类会读取这些属性并进行配置。

### 3. 自定义条件注解

你可以使用 `@Conditional` 注解定义自己的生效条件逻辑。

---

## ✅ 五、调试和可视化自动配置

### 1. 使用 Spring Boot Actuator 查看自动配置状态

添加依赖：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

启用自动配置报告：

```yaml
management:
  endpoints:
    web:
      exposure:
        include: conditions
```

访问地址：

```
http://localhost:8080/actuator/conditions
```

### 2. 命令查看依赖关系

```bash
mvn dependency:tree
```

---

## ✅ 六、Starter 模块机制

Spring Boot 通过 starter 模块引入依赖和触发自动配置。

常见 starter：

| Starter 名称 | 功能 |
|--------------|------|
| `spring-boot-starter-web` | 引入 Spring MVC、Tomcat 等依赖 |
| `spring-boot-starter-data-jpa` | 引入 Hibernate、JPA 支持 |
| `spring-boot-starter-security` | 引入安全认证模块 |

---

## ✅ 七、总结

- 自动配置通过 `@EnableAutoConfiguration` 启动；
- 借助条件注解实现按需配置；
- 用户可通过定义 Bean 或配置文件灵活控制；
- 提高开发效率，降低配置成本，是 Spring Boot 的核心优势。

