
# 🌐 Spring Boot 全局异常处理器使用笔记

## 📌 1. 什么是全局异常处理？

全局异常处理是指在 Spring Boot 项目中，对控制器（Controller）中出现的异常进行统一处理，避免每个方法都写重复的 try-catch，提高代码整洁性和可维护性。

---

## 🧱 2. 基本用法

### ✅ 创建一个全局异常处理类

```java
package com.example.exception;

import com.example.pojo.Result;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public Result handleException(Exception e) {
        e.printStackTrace(); // 打印堆栈，便于开发调试
        String message = StringUtils.hasLength(e.getMessage()) ? e.getMessage() : "操作失败";
        return Result.error(message); // 自定义错误返回格式
    }
}
```

---

## 🧠 3. 注解说明

| 注解 | 说明 |
|------|------|
| `@RestControllerAdvice` | 标识该类为全局异常处理类，返回值会自动以 JSON 格式输出（等同于 `@ControllerAdvice + @ResponseBody`） |
| `@ExceptionHandler(Exception.class)` | 指定当前方法处理哪种类型的异常（此处为所有异常） |

---

## 🎯 4. 常见异常处理扩展

### ✅ 处理空指针异常

```java
@ExceptionHandler(NullPointerException.class)
public Result handleNullPointer(NullPointerException e) {
    return Result.error("空指针异常，请联系管理员！");
}
```

### ✅ 处理参数校验异常（例如 @Valid）

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public Result handleValidation(MethodArgumentNotValidException e) {
    String msg = e.getBindingResult().getFieldError().getDefaultMessage();
    return Result.error("参数校验失败：" + msg);
}
```

---

## 🔍 5. 能捕获哪些异常？

| 异常来源 | 是否能捕获 | 说明 |
|----------|-------------|------|
| Controller 方法中抛出的异常 | ✅ | 最常见使用场景 |
| Service 中抛出但未处理的异常 | ✅ | 会传递到 Controller，再由全局处理器捕获 |
| Service 中 try-catch 后自己处理了异常 | ❌ | 全局处理器无感知 |
| 异常发生在拦截器、过滤器、异步线程中 | ❌ | 需单独处理 |

---

## 🚫 注意事项

- 如果异常被提前 catch 并处理了，全局异常处理器无法捕获。
- `@RestControllerAdvice` 默认返回 JSON；如果需要返回页面，应使用 `@ControllerAdvice`。
- 建议抛出业务异常时，提供清晰的 message 信息。

---

## 📦 6. 示例 Result 返回类（参考）

```java
public class Result {
    private Integer code;
    private String message;
    private Object data;

    public static Result error(String message) {
        Result r = new Result();
        r.setCode(500);
        r.setMessage(message);
        r.setData(null);
        return r;
    }

    // 省略 getter/setter 和其他静态方法
}
```

---

## 📚 7. 小结

- ✅ 全局异常处理器能让你的 Spring Boot 项目更健壮、易维护。
- ✅ 异常不再零散处理，而是统一封装、统一返回。
- ✅ 建议根据项目需求，自定义更多 `@ExceptionHandler` 方法。

---

## 🛠 推荐组合使用

- `@RestControllerAdvice`
- `@ExceptionHandler`
- `Result` 统一响应封装
- 配合参数校验（如 `@Valid`、`@Validated`）
