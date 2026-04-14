# 数据库设计（P2b）

## 0. 设计基线
- 数据库：MySQL 8.0+
- 命名：`snake_case`
- 主键：统一 `id`（`bigint unsigned`）
- 金额字段：统一 `decimal(18,4)`（本MVP暂无金额业务，预留规范）
- 审计字段（业务表强制）：`created_at`、`created_by`、`updated_at`、`updated_by`
- 软删除字段（业务表强制）：`is_deleted`、`deleted_at`
- 分页策略：
  - 常规列表：页码分页
  - 大表（>100万）：游标分页（`id > last_id`）
- 加密策略：PII（手机号、证件号）采用 AES-256-GCM（应用层）

---

## 1. user_account
- 业务用途：系统登录账户与人员主档。
- 字段
  - `id` bigint unsigned not null default 0 comment '主键ID'
  - `username` varchar(64) not null default '' comment '登录账号（唯一）'
  - `password_hash` varchar(255) not null default '' comment '密码哈希'
  - `password_salt` varchar(128) not null default '' comment '密码盐'
  - `display_name` varchar(64) not null default '' comment '姓名'
  - `mobile_encrypted` varchar(255) not null default '' comment '手机号密文'
  - `id_no_encrypted` varchar(255) not null default '' comment '证件号密文'
  - `status` tinyint not null default 1 comment '状态:1启用0禁用'
  - 审计与软删字段同基线
- 索引策略：`uk_username`；`idx_status`；`idx_created_at`
- 关系：被 `user_role_rel`、`exam_session_user`、`answer_sheet` 等引用；不设物理外键（高并发与迁移弹性）
- 性能备注：预计10万用户；按 `id` 游标分页
- 安全备注：手机号/证件号密文存储；密码不可逆哈希

## 2. role_dict（字典表）
- 业务用途：角色字典（考生/出题人/阅卷人/管理员/系统管理员）。
- 字段：`id`、`role_code`、`role_name`、`sort_no`、`status` + 审计软删
- 索引策略：`uk_role_code`
- 关系：与 `user_role_rel` 逻辑关联
- 性能备注：小表，常驻缓存
- 安全备注：无PII

## 3. user_role_rel
- 业务用途：用户与角色多对多映射。
- 字段：`id`、`user_id`、`role_id` + 审计软删
- 索引策略：`uk_user_role(user_id,role_id,is_deleted)`；`idx_role_id`
- 关系：关联 `user_account`、`role_dict`
- 性能备注：预计50万行；按 `user_id` 查询高频
- 安全备注：权限变更需审计

## 4. org_unit
- 业务用途：组织架构（部门/小组）
- 字段：`id`、`org_code`、`org_name`、`parent_id`、`org_level`、`status` + 审计软删
- 索引策略：`uk_org_code`；`idx_parent_id`
- 关系：被 `user_org_rel` 引用
- 性能备注：树形查询按 `parent_id` 分层
- 安全备注：无PII

## 5. user_org_rel
- 业务用途：用户与组织归属关系
- 字段：`id`、`user_id`、`org_id`、`is_primary` + 审计软删
- 索引策略：`uk_user_org(user_id,org_id,is_deleted)`；`idx_org_id`
- 关系：关联 `user_account`、`org_unit`
- 性能备注：预计30万行
- 安全备注：组织变更全量审计

## 6. question_bank
- 业务用途：题目主表
- 字段：`id`、`question_type`、`difficulty`、`stem`、`analysis_text`、`score`、`status` + 审计软删
- 索引策略：`idx_type_difficulty`；`idx_status`
- 关系：被 `question_option`、`question_tag_rel`、`exam_paper_question` 引用
- 性能备注：目标20万题；按 `id` 游标分页
- 安全备注：内容按业务分级访问

## 7. question_option
- 业务用途：选择题选项
- 字段：`id`、`question_id`、`option_key`、`option_text`、`is_correct`、`sort_no` + 审计软删
- 索引策略：`idx_question_id`
- 关系：关联 `question_bank`
- 性能备注：按 `question_id` 批量查询
- 安全备注：正确答案字段仅阅卷服务可见

## 8. tag_dict（字典表）
- 业务用途：知识点/标签字典
- 字段：`id`、`tag_code`、`tag_name`、`status` + 审计软删
- 索引策略：`uk_tag_code`
- 关系：与 `question_tag_rel` 逻辑关联
- 性能备注：小表缓存
- 安全备注：无PII

## 9. question_tag_rel
- 业务用途：题目与标签多对多关系
- 字段：`id`、`question_id`、`tag_id` + 审计软删
- 索引策略：`uk_question_tag(question_id,tag_id,is_deleted)`；`idx_tag_id`
- 关系：关联 `question_bank`、`tag_dict`
- 性能备注：预计100万+，禁止 offset 深翻页
- 安全备注：无PII

## 10. exam_template
- 业务用途：考试模板与规则主表
- 字段：`id`、`template_name`、`pass_score`、`duration_minutes`、`start_time`、`end_time`、`version_no`、`status` + 审计软删
- 索引策略：`idx_status_time(status,start_time,end_time)`；`uk_template_version(template_name,version_no,is_deleted)`
- 关系：被 `exam_template_rule`、`exam_session` 引用
- 性能备注：发布时间窗过滤高频
- 安全备注：发布后冻结

## 11. exam_template_rule
- 业务用途：固定题+随机题组卷规则
- 字段：`id`、`template_id`、`rule_type`、`question_type`、`difficulty`、`question_count`、`fixed_question_id`、`sort_no` + 审计软删
- 索引策略：`idx_template_id`；`idx_rule_type`
- 关系：关联 `exam_template`、`question_bank`
- 性能备注：模板发布前校验规则完整性
- 安全备注：无PII

## 12. exam_session
- 业务用途：考试场次实例
- 字段：`id`、`template_id`、`session_name`、`session_status`、`publish_time`、`result_publish_time` + 审计软删
- 索引策略：`idx_template_id`；`idx_session_status`
- 关系：被 `exam_session_user`、`answer_sheet` 引用
- 性能备注：预计10万场次
- 安全备注：状态流转需审计

## 13. exam_session_user
- 业务用途：场次与考生分配
- 字段：`id`、`session_id`、`user_id`、`assign_status` + 审计软删
- 索引策略：`uk_session_user(session_id,user_id,is_deleted)`；`idx_user_id`
- 关系：关联 `exam_session`、`user_account`
- 性能备注：预计500万+，按 `session_id` 与 `id` 游标分页
- 安全备注：权限校验避免越权开考

## 14. exam_paper_question
- 业务用途：考生试卷题目快照
- 字段：`id`、`session_id`、`user_id`、`question_id`、`question_order`、`score` + 审计软删
- 索引策略：`idx_session_user(session_id,user_id)`；`idx_question_id`
- 关系：关联 `exam_session`、`user_account`、`question_bank`
- 性能备注：预计千万级；按 `session_id+user_id+id` 游标读取
- 安全备注：快照只读，防篡改

## 15. answer_sheet
- 业务用途：答卷主表
- 字段：`id`、`session_id`、`user_id`、`sheet_status`、`start_time`、`submit_time`、`auto_score`、`review_score`、`final_score` + 审计软删
- 索引策略：`uk_session_user(session_id,user_id,is_deleted)`；`idx_sheet_status`
- 关系：被 `answer_item`、`review_task`、`result_publish` 引用
- 性能备注：预计500万+；禁止 offset 深翻页
- 安全备注：分数变更全留痕

## 16. answer_item
- 业务用途：答卷明细（逐题作答）
- 字段：`id`、`sheet_id`、`question_id`、`answer_content`、`is_correct`、`score` + 审计软删
- 索引策略：`idx_sheet_id`；`idx_question_id`
- 关系：关联 `answer_sheet`、`question_bank`
- 性能备注：预计亿级；按 `sheet_id+id` 游标分页；月归档
- 安全备注：主观题答案按敏感级别控制

## 17. anti_cheat_event
- 业务用途：防作弊事件（切屏/复制/超时等）
- 字段：`id`、`session_id`、`user_id`、`event_type`、`event_detail`、`event_time` + 审计软删
- 索引策略：`idx_session_user_time(session_id,user_id,event_time)`
- 关系：关联 `exam_session`、`user_account`
- 性能备注：预计千万级；30天热数据，之后归档
- 安全备注：事件明细用于审计

## 18. review_task
- 业务用途：主观题复核任务
- 字段：`id`、`sheet_id`、`reviewer_id`、`task_status`、`review_comment`、`review_time` + 审计软删
- 索引策略：`idx_reviewer_status(reviewer_id,task_status)`；`idx_sheet_id`
- 关系：关联 `answer_sheet`、`user_account`
- 性能备注：复核队列分页按 `id` 游标
- 安全备注：复核意见不可物理删除

## 19. result_publish
- 业务用途：成绩发布记录
- 字段：`id`、`session_id`、`sheet_id`、`publish_status`、`published_at`、`publisher_id` + 审计软删
- 索引策略：`idx_session_id`；`uk_sheet_publish(sheet_id,is_deleted)`
- 关系：关联 `exam_session`、`answer_sheet`
- 性能备注：发布后只追加版本
- 安全备注：撤回需记录原因

## 20. certificate
- 业务用途：电子证书
- 字段：`id`、`sheet_id`、`certificate_no`、`certificate_status`、`issued_at`、`file_url` + 审计软删
- 索引策略：`uk_certificate_no`；`idx_sheet_id`
- 关系：关联 `answer_sheet`
- 性能备注：证书查询按编号唯一检索
- 安全备注：下载链接需鉴权

## 21. audit_event
- 业务用途：审计日志
- 字段：`id`、`event_domain`、`event_action`、`operator_id`、`target_id`、`event_payload`、`event_time` + 审计软删
- 索引策略：`idx_domain_time(event_domain,event_time)`；`idx_operator_time(operator_id,event_time)`
- 关系：逻辑关联全部业务对象
- 性能备注：预计亿级；按月分区
- 安全备注：不可篡改，保留5年

## 22. export_job
- 业务用途：导出任务
- 字段：`id`、`job_type`、`job_status`、`request_payload`、`requested_by`、`requested_at`、`finished_at` + 审计软删
- 索引策略：`idx_status_time(job_status,requested_at)`
- 关系：关联 `export_file`
- 性能备注：异步任务队列
- 安全备注：请求参数审计留痕

## 23. export_file
- 业务用途：导出文件元数据
- 字段：`id`、`job_id`、`file_name`、`file_url`、`file_size`、`expire_at` + 审计软删
- 索引策略：`idx_job_id`；`idx_expire_at`
- 关系：关联 `export_job`
- 性能备注：到期清理
- 安全备注：下载需鉴权

## 24. 大表与归档策略
- `answer_item`、`audit_event`、`exam_paper_question`、`exam_session_user` 预计>100万。
- 策略：
  1. 查询统一游标分页：`where id > :last_id order by id asc limit :limit`
  2. `answer_item`、`audit_event` 按月分区（`p202604`）
  3. 热数据 6 个月，冷数据归档到历史库
  4. 归档后通过离线导出检索
