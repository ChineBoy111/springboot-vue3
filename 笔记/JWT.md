
# 📘 JWT Token 认证机制简介

## 📌 什么是 JWT？

JWT（JSON Web Token）是一种基于 JSON 格式的、用于 **身份认证和信息传递** 的令牌机制，常用于前后端分离系统中的用户登录认证。

---

## 🔐 JWT 的基本作用

- **用户身份认证**：登录成功后，生成 JWT，客户端每次请求都带上它，服务器通过验证 JWT 判断用户身份。
- **信息传递**：JWT 还可以在 Payload 中携带非敏感用户信息（如用户名、权限等）。

---

## 🧱 JWT 的结构

JWT 是由三段组成的字符串，格式如下：

```
Header.Payload.Signature
```

例如：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJ1c2VybmFtZSI6IndkeCIsImlhdCI6MTY5NTYxMjM0NX0.
hT8W6cVK3w0Qazje1e1V1vAtcBgJfTb8VbEBYzqKUIg
```

| 部分 | 名称     | 内容说明                       |
|------|----------|--------------------------------|
| 第一部分 | Header   | 指定签名算法（如 HS256）和类型（JWT） |
| 第二部分 | Payload  | 携带用户信息（如 ID、用户名、权限等） |
| 第三部分 | Signature | 用密钥对前两部分进行加密，防止篡改     |

---

## 🚀 JWT 的使用流程

1. 用户登录，服务器验证账号密码
2. 登录成功后，服务器生成 JWT 并返回给客户端
3. 客户端保存 JWT（一般保存在 LocalStorage 或 Cookie）
4. 每次请求时客户端在请求头中携带 JWT（如 `Authorization: Bearer xxx`）
5. 服务端验证 JWT：
   - 是否未过期？
   - 签名是否正确？
   - 是否在黑名单中（如果需要）？
6. 验证通过则继续处理请求，否则返回 401 未授权

---

## ✅ JWT 的优点

- 无需服务器保存登录状态（**无状态认证**）
- 前后端分离支持好（可跨域）
- JSON 格式，跨语言平台通用
- 可自定义 Payload 携带所需信息（如用户名、权限等）

---

## ⚠️ JWT 的缺点与注意事项

| 缺点             | 说明                                     |
|------------------|------------------------------------------|
| 无法主动失效     | 用户退出登录，JWT 依旧有效，除非用黑名单机制处理 |
| Payload 可被解码 | 不能放敏感信息（如密码）                     |
| 长度较大         | 比 Session ID 更长，可能增加网络传输成本         |

---

## 📦 示例 Payload

```json
{
  "sub": "wdx",
  "role": "admin",
  "iat": 1718217600,
  "exp": 1718224800
}
```

说明：

- `sub`：主题（用户标识）
- `role`：权限
- `iat`：签发时间（issued at）
- `exp`：过期时间（expiration time）

---

## 📌 相关技术点（Spring Boot 中常用）

- 登录接口：验证成功后生成 JWT
- 使用 `jjwt`、`java-jwt` 等库生成和解析令牌
- 使用拦截器（如 `OncePerRequestFilter`）统一校验 JWT
- 设置请求头：`Authorization: Bearer token`
- 添加 token 过期时间和刷新机制
- 可设计 JWT 黑名单策略提升安全性

---

## 📘 总结

JWT 是现代 Web 应用中非常流行的认证机制，适用于前后端分离系统和微服务架构。但它不是银弹，设计时应注意信息安全与过期控制等问题。

---

## 📎 推荐扩展

- Spring Security + JWT 实现完整登录认证流程
- JWT 刷新机制（Refresh Token）
- Redis 实现 Token 黑名单
