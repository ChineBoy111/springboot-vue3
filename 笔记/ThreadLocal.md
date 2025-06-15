
# 🧵 ThreadLocal 在 Spring Boot + Vue 项目中的应用

## ✅ 什么是 ThreadLocal？

`ThreadLocal` 是 Java 中用于实现 **线程本地变量（Thread-Local Variable）** 的工具类：

- 每个线程都有独立的变量副本，不会互相干扰
- 在 Web 项目中，常用于存储与当前请求线程相关的数据（如当前登录用户）

---

## 🧩 应用场景：当前用户信息共享

在 Spring Boot + Vue 项目中，前端 Vue 登录后携带 token 请求接口，后端解析 token 获取当前用户。

### ✨ 目标

在 Controller、Service 中 **随时获取当前用户信息**，无需层层传参。

---

## 🔧 实现步骤

### 1️⃣ 创建工具类 UserHolder

```java
public class UserHolder {
    private static final ThreadLocal<User> userThreadLocal = new ThreadLocal<>();

    public static void set(User user) {
        userThreadLocal.set(user);
    }

    public static User get() {
        return userThreadLocal.get();
    }

    public static void remove() {
        userThreadLocal.remove(); // 防止内存泄漏
    }
}
```

---

### 2️⃣ 在拦截器中解析 token 并存入 ThreadLocal

```java
@Component
public class LoginInterceptor implements HandlerInterceptor {

    @Autowired
    private UserService userService;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        String token = request.getHeader("Authorization");

        if (token != null && !token.isEmpty()) {
            User user = userService.getUserByToken(token);
            if (user != null) {
                UserHolder.set(user); // 存入 ThreadLocal
            }
        }
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        UserHolder.remove(); // 防止内存泄漏
    }
}
```

---

### 3️⃣ 在业务逻辑中获取当前用户

```java
@Service
public class OrderService {
    public void createOrder() {
        User currentUser = UserHolder.get();
        Long userId = currentUser.getId();
        // 继续业务逻辑
    }
}
```

---

## ✅ 优点总结

| 优点 | 描述 |
|------|------|
| ✅ 避免重复传参 | 不需要在 Controller → Service → DAO 中层层传递用户信息 |
| ✅ 与请求线程绑定 | 每个请求线程维护一份独立数据，互不干扰 |
| ✅ 解耦业务代码 | 不必每次都重新解析 token |

---

## ⚠️ 注意事项

| 问题             | 描述 |
|------------------|------|
| ⚠️ 内存泄漏风险   | 必须在请求完成后 `remove()`，避免线程池重用时泄漏数据 |
| ⚠️ 不适用异步场景 | 如线程池、异步任务中不适用，ThreadLocal 是线程绑定的 |

---

## 🔚 总结

- `ThreadLocal` 是 Spring Boot 项目中共享“当前用户”等上下文信息的理想选择
- 配合拦截器+JWT 可实现安全、简洁的用户认证机制
- 使用完毕必须调用 `remove()` 清理资源，确保线程安全

---

## 📘 延伸阅读

- JWT 与用户身份认证
- Spring Security 中的 `SecurityContextHolder` 也是 ThreadLocal 的典型应用
