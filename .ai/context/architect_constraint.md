# 架构约束配置

## 技术栈锁定

### 后端框架

- Java 17
- Spring Boot 3.2.x
- Spring Security 6.x
- MyBatis Plus 3.5.x (ORM)
- MySQL 8.0+ (数据库)

### 前端框架

- Vue 3.4+ (Composition API)
- TypeScript 5.3+
- Pinia (状态管理)
- Vite (构建工具)

### 缓存与消息

- Redis 7 (会话存储、分布式锁、防作弊计数)

### 认证授权

- JWT (JSON Web Token)
- Spring Security + JWT (jjwt库)

## 禁用依赖

- 禁止使用第三方前端UI组件库（需自行设计组件）
- 禁止使用MyBatis Plus之外的数据访问方式
- 禁止引入WebSocket实时通信（MVP阶段）

## 部署约束

- 支持容器化部署（Docker）
- 支持反向代理（Nginx）
- 数据库连接池最大连接数：100

## 性能约束

- API响应时间：P95 < 500ms
- 数据库查询：单次 < 100ms
- 并发用户：200人同时在线考试
