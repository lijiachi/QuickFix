# QuickFix - 智能校园维修管理系统

面向校园场景的在线维修协作平台，涵盖 **学生报修 → 师傅接单处理 → 管理员调度管理** 的完整业务闭环。

## 项目结构

```
project/
├── quickfix/       # 后端 (Spring Boot 3 + Java 17)
└── quickfix-ui/    # 前端 (Vue 3 + Vite + Element Plus)
```

## 技术栈

### 后端
- Spring Boot 3.4.1
- Spring Security + JWT (jjwt 0.12.6)
- MyBatis-Plus 3.5.7
- MySQL 8.x

### 前端
- Vue 3
- Vite 8
- Pinia (状态管理)
- Vue Router 4
- Element Plus
- Axios

## 角色与功能

| 角色 | 功能 |
|------|------|
| **admin (管理员)** | 用户管理 · 维修单全量查看 · 任意状态切换 |
| **maintenance (维修师傅)** | 维修单全量查看 · 一键"完成"操作 |
| **student (学生)** | 仅查看本人报修单 · 提交新报修 · 一键"取消"未完结的报修 |

维修单状态流转：`待处理(0) → 处理中(1) → 已完成(2)`，或直接 `已取消(3)`。

## 快速开始

### 环境要求

- JDK 17+
- Node.js 22+
- MySQL 8.x

### 1. 启动后端

```bash
cd quickfix

# 修改 application.properties 中的数据库连接（如账号密码）
# 默认: root / 123456, 数据库 fix（自动创建）

./mvnw spring-boot:run
```

后端默认运行在 **http://localhost:8081**

### 2. 启动前端

```bash
cd quickfix-ui
npm install
npm run dev
```

前端默认运行在 **http://localhost:5173**，Vite 代理已配置 `/api` → `http://localhost:8081`。

### 默认账号

| 用户名 | 密码 | 角色 |
|--------|------|------|
| admin | admin123 | 管理员 |
| fixer | fixer123 | 维修师傅 |
| student | student123 | 学生 |

## 核心设计

### 声明式 RBAC 权限体系

后端通过 `@PreAuthorize` 注解在方法级精确控制接口访问：

```java
@GetMapping("/list/{userId}")
@PreAuthorize("isAuthenticated()")
public Result<?> listByUser(@PathVariable Long userId) { ... }

@GetMapping("/list")
@PreAuthorize("hasRole('admin') or hasRole('maintenance')")
public Result<?> list() { ... }
```

前端通过 **Pinia store + Vue Router 路由守卫 + 按钮级 `v-if`** 三层收敛，与后端鉴权保持双端一致。

### 统一异常处理

`@RestControllerAdvice` 将认证失败、权限不足、业务异常统一转换为 `Result<T>` 规范响应：

```json
{ "code": 401, "message": "未登录或 token 已过期", "data": null }
{ "code": 403, "message": "无权限访问", "data": null }
{ "code": 400, "message": "用户名或密码错误", "data": null }
```

### JWT 无状态认证

- 自定义 `JwtAuthenticationFilter` 从请求头提取 `Authorization: Bearer <token>`
- `JwtTokenUtil` 负责签发 / 解析 / 校验
- 密码使用 BCrypt 加密存储
- Token 有效期 7200 秒（2 小时）

## API 概览

| 方法 | 路径 | 权限 | 说明 |
|------|------|------|------|
| POST | `/login` | 公开 | 账号密码登录 |
| GET | `/maintenance/order/list` | admin / maintenance | 全部维修单 |
| GET | `/maintenance/order/list/{userId}` | 已登录 | 指定用户维修单 |
| POST | `/maintenance/order/add` | 已登录 | 新增维修单 |
| PUT | `/maintenance/order/changeStatus` | 已登录 | 更新维修单状态 |
| GET | `/user/list` | admin | 用户列表 |
| POST | `/user/add` | admin | 新增用户 |

## License

MIT
