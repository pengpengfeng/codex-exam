# API契约骨架（P2a）

## 1. 协议
- 传输协议：HTTPS
- 数据格式：`application/json; charset=utf-8`
- 字符编码：UTF-8
- 时间格式：ISO-8601（UTC+8 展示，存储 UTC）

## 2. 命名规范
- URI 使用 kebab-case：`/api/v1/exam-sessions`
- JSON 字段使用 camelCase：`examSessionId`
- 布尔字段前缀：`is/has/can`
- 列表接口统一分页参数：`pageNo`、`pageSize`、`cursor`（二选一模式）

## 3. 认证方式
- 登录接口：账号密码登录，成功后签发 JWT AccessToken + RefreshToken
- 鉴权：`Authorization: Bearer <token>`
- 权限模型：RBAC（role + permission）

## 4. 错误码方案
- 业务码结构：`EXAM-<DOMAIN>-<NNN>`（例：`EXAM-AUTH-001`）
- 典型错误域：
  - AUTH（认证鉴权）
  - EXAM（考试执行）
  - PAPER（组卷题库）
  - GRADE（评分复核）
  - CERT（证书结果）
  - AUDIT（审计导出）

## 5. 响应包装结构
```json
{
  "code": "0",
  "message": "success",
  "traceId": "string",
  "data": {},
  "timestamp": "2026-04-13T00:00:00Z"
}
```
- `code="0"` 表示成功，非0为失败
- 所有失败响应需包含可追溯 `traceId`

## 6. 分页模式
- 普通列表：页码分页（`pageNo` + `pageSize`）
- 大数据量导出/审计：游标分页（`cursor` + `limit`）
- 单页上限：`pageSize <= 100`

## 7. 接口清单（Schema 待补全）

### 7.1 认证与用户
1. `POST /api/v1/auth/login`
   - 请求体 Schema: `[TBD]`
   - 成功响应 Schema: `[TBD]`
   - 错误响应 Schema: `[TBD]`
2. `POST /api/v1/auth/refresh`
   - 请求体 Schema: `[TBD]`
   - 成功响应 Schema: `[TBD]`
   - 错误响应 Schema: `[TBD]`
3. `GET /api/v1/users/me`
   - 请求参数 Schema: `[TBD]`
   - 成功响应 Schema: `[TBD]`
   - 错误响应 Schema: `[TBD]`

### 7.2 题库与模板
4. `POST /api/v1/questions`
   - 请求体 Schema: `[TBD]`
   - 成功响应 Schema: `[TBD]`
   - 错误响应 Schema: `[TBD]`
5. `GET /api/v1/questions`
   - 请求参数 Schema: `[TBD]`
   - 成功响应 Schema: `[TBD]`
   - 错误响应 Schema: `[TBD]`
6. `POST /api/v1/exam-templates`
   - 请求体 Schema: `[TBD]`
   - 成功响应 Schema: `[TBD]`
   - 错误响应 Schema: `[TBD]`
7. `POST /api/v1/exam-templates/{templateId}/publish`
   - 请求体 Schema: `[TBD]`
   - 成功响应 Schema: `[TBD]`
   - 错误响应 Schema: `[TBD]`

### 7.3 考试执行
8. `POST /api/v1/exam-sessions/{sessionId}/start`
   - 请求体 Schema: `[TBD]`
   - 成功响应 Schema: `[TBD]`
   - 错误响应 Schema: `[TBD]`
9. `POST /api/v1/exam-sessions/{sessionId}/answers/save`
   - 请求体 Schema: `[TBD]`
   - 成功响应 Schema: `[TBD]`
   - 错误响应 Schema: `[TBD]`
10. `POST /api/v1/exam-sessions/{sessionId}/submit`
    - 请求体 Schema: `[TBD]`
    - 成功响应 Schema: `[TBD]`
    - 错误响应 Schema: `[TBD]`

### 7.4 评分与发布
11. `POST /api/v1/grading/review-tasks/{taskId}/submit`
    - 请求体 Schema: `[TBD]`
    - 成功响应 Schema: `[TBD]`
    - 错误响应 Schema: `[TBD]`
12. `POST /api/v1/results/{examId}/publish`
    - 请求体 Schema: `[TBD]`
    - 成功响应 Schema: `[TBD]`
    - 错误响应 Schema: `[TBD]`
13. `GET /api/v1/certificates/{certificateNo}`
    - 请求参数 Schema: `[TBD]`
    - 成功响应 Schema: `[TBD]`
    - 错误响应 Schema: `[TBD]`

### 7.5 报表与审计
14. `GET /api/v1/reports/exams/{examId}/summary`
    - 请求参数 Schema: `[TBD]`
    - 成功响应 Schema: `[TBD]`
    - 错误响应 Schema: `[TBD]`
15. `POST /api/v1/audit/exports`
    - 请求体 Schema: `[TBD]`
    - 成功响应 Schema: `[TBD]`
    - 错误响应 Schema: `[TBD]`
16. `GET /api/v1/audit/exports/{jobId}`
    - 请求参数 Schema: `[TBD]`
    - 成功响应 Schema: `[TBD]`
    - 错误响应 Schema: `[TBD]`
