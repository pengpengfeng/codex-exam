## 1. 架构影响分析
- 范围承接：MVP包含账号密码登录、混合组卷、自动+人工阅卷、证书与5年审计留存。
- 关键冲突：需求目标“并发≤500考生”与约束“200人同时在线考试”不一致；需按“单实例200、双实例500”进行容量分层。
- 约束落地：后端固定 Java17 + Spring Boot 3.2 + Spring Security + MyBatis Plus；前端 Vue3+TS+Pinia；缓存 Redis7；禁用 WebSocket 与第三方 UI 库。
- 交互模式：考试过程采用轮询+短请求（非实时推送），满足禁用 WebSocket 约束。
- 设计原则：高内聚分层、读写分离（逻辑层面）、审计优先、可追溯优先。

## 2. 逻辑架构设计
### 2.1 模块与职责
1. auth-service（认证与会话）
   - 账号密码登录、JWT签发/刷新、权限装载。
   - 维护登录风控计数（Redis）。
2. user-org-service（用户组织）
   - 维护考生、出题人、阅卷人、管理员角色映射。
   - 提供组织维度授权与数据过滤。
3. question-bank-service（题库）
   - 题目、题型、难度、知识点标签管理。
   - 题库版本快照供组卷使用。
4. exam-template-service（考试模板）
   - 固定题+随机题混合规则、及格线、时间窗配置。
   - 模板版本化与发布冻结。
5. exam-runtime-service（考试执行）
   - 开考、答题暂存、交卷、超时自动交卷。
   - 防作弊事件记录（切屏、复制、超时）。
6. grading-service（评分）
   - 客观题自动判分、主观题复核任务流转。
   - 成绩汇总、发布前校验。
7. result-certificate-service（结果与证书）
   - 成绩发布、错题解析、电子证书生成与编号。
   - 证书下载与验真（内网）。
8. report-audit-service（报表审计）
   - 完成率/通过率/均分报表。
   - 审计导出与导出留痕。
9. admin-config-service（系统配置）
   - 密码策略、考试策略、参数中心。
   - 字典配置与系统开关。

### 2.2 依赖关系
- 前端 BFF/API Gateway → auth-service（鉴权）→ 各业务服务。
- exam-runtime-service 依赖 exam-template-service、question-bank-service 拉取试卷快照。
- grading-service 依赖 exam-runtime-service 作答数据与 question-bank-service 标准答案。
- result-certificate-service 依赖 grading-service 最终成绩。
- report-audit-service 汇聚 runtime/grading/result 的审计事件。
- 全服务共享 MySQL（逻辑分库分表预留）+ Redis（会话、锁、计数）。

### 2.3 数据流方向
1. 登录流：账号密码 → auth-service → JWT/RefreshToken → 前端持有。
2. 开考流：考生开考 → runtime读取模板快照 → 生成考生试卷实例。
3. 作答流：前端定时保存 → runtime落库 + Redis防抖。
4. 交卷流：runtime提交 → grading自动判分 → 主观题复核 → 成绩汇总。
5. 发布流：管理员发布 → result生成解析/证书 → report写审计事件。
6. 审计流：业务事件 → audit日志表 → 导出任务异步生成文件。

## 3. 数据与状态设计
### 3.1 核心实体变更
- 用户域：user、role、user_role、org_unit。
- 题库域：question、question_option、knowledge_tag、question_version。
- 考试域：exam_template、exam_template_rule、exam_session、exam_paper_instance。
- 作答域：answer_sheet、answer_item、anti_cheat_event。
- 评分域：grading_record、subjective_review_task、score_summary。
- 结果域：result_publish、certificate。
- 审计域：audit_event、export_job、export_file。

### 3.2 状态机（关键）
- exam_session：DRAFT → PUBLISHED → IN_PROGRESS → ENDED → ARCHIVED。
- answer_sheet：NOT_STARTED → IN_PROGRESS → SUBMITTED → AUTO_GRADED → REVIEWED → FINALIZED。
- subjective_review_task：PENDING → IN_REVIEW → REVIEWED → REOPENED/CLOSED。
- result_publish：PREPARED → PUBLISHED → REVOKED（需审计原因）。

### 3.3 一致性风险与策略
- 风险1：自动判分与人工复核并发写冲突。
  - 策略：score_summary 使用版本号（optimistic lock）+ 幂等更新键（exam_id+user_id）。
- 风险2：超时自动交卷与手工交卷竞态。
  - 策略：交卷操作加分布式锁（Redis key: submit:{session}:{user}）+ 状态机单向约束。
- 风险3：成绩发布后复核回滚。
  - 策略：发布采用“冻结快照”，回滚仅可产生新版本，不覆盖旧版本。
- 风险4：审计导出大查询影响在线考试。
  - 策略：导出异步任务+只读副本（后续阶段）+ 游标分页。

## 4. 非功能性分析（量化）
1. 性能
   - 在线考试核心接口（开考、保存答案、交卷）P95 < 500ms。
   - 非核心查询接口 P95 < 800ms；导出任务进入队列 < 30s。
   - 单次数据库查询 < 100ms（复杂报表除外，走异步）。
2. 并发
   - 单实例稳定支撑 200 同时在线考试（满足约束）。
   - 双实例+Nginx 负载可支撑目标 500 并发考生（需压测验证）。
3. 权限安全
   - JWT 访问令牌有效期 30min，刷新令牌 7d。
   - 关键管理接口强制 RBAC + 审计日志 100%留痕。
   - 密码策略最少12位，90天轮换，连续5次失败锁定15分钟。
4. 可用性
   - 核心服务可用性目标 99.9%（月）。
   - RPO ≤ 15min，RTO ≤ 30min（数据库备份+恢复演练）。
5. 可维护性
   - 统一错误码、统一响应包装、统一链路追踪ID。
   - 核心业务（登录/开考/交卷/评分/发布）必须有自动化回归。

## 5. 风险与权衡
| 风险 | 概率 | 影响 | 缓解措施 |
|---|---|---|---|
| 并发目标500与约束200冲突 | 高 | 高 | 采用双实例容量设计并先做200基线验收，再做500压测扩容 |
| 无WebSocket导致反作弊实时性下降 | 中 | 中 | 采用5秒轮询+事件阈值告警，MVP先满足留痕与追溯 |
| 5年留存带来存储膨胀 | 高 | 中 | 冷热分层+按月归档+导出文件生命周期策略 |
| 主观题复核积压影响发布时间 | 中 | 高 | 复核SLA与批量分配机制，超时自动升级提醒 |
| 第三方UI库禁用增加前端工期 | 中 | 中 | 建立内部基础组件库（按钮/表单/表格）优先复用 |

## 6. 替代方案评估与最终决策
### 方案A（采纳）：模块化单体 + 分层服务
- 最终决策：采用方案A（模块化单体 + 分层服务）作为MVP唯一实施架构。

- 特点：单应用多模块，按领域拆包，统一部署，Redis+MySQL。
- 采纳理由：MVP阶段团队规模与并发规模可控，交付速度快，运维复杂度低。

### 方案B（拒绝）：微服务拆分（独立部署多个服务）
- 特点：每领域独立服务与数据库、网关统一路由。
- 拒绝理由：当前MVP（内网、并发规模有限）下治理与运维成本过高，发布链路复杂，收益不足。


## 7. 落地执行约束（按方案A）
- 部署形态：单应用容器化部署，Nginx反向代理，按环境配置多实例扩展。
- 代码组织：按领域模块拆包（auth/exam/grading/report），统一分层（controller/service/repository）。
- 数据访问：仅使用 MyBatis Plus；禁止引入其他数据访问方案。
- 通信策略：服务内同步调用 + 异步任务队列（导出/证书），不使用 WebSocket。
- 演进路径：当并发长期>500且团队规模扩大，再评估拆分微服务。
