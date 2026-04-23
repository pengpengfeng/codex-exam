# UI设计草稿（P3）

## 1. 设计层
- 目标：企业内网在线考试系统，突出“快、稳、清晰”。
- 信息架构：
  1) 登录页
  2) 考生端：考试列表 / 答题页 / 结果页
  3) 管理端：题库管理 / 模板配置 / 场次发布 / 阅卷中心 / 报表审计
- 导航：
  - 考生端：顶部状态栏 + 主内容
  - 管理端：左侧导航 + 顶部操作栏 + 内容区

## 2. UI输出（组件状态显式）
### 2.1 Button
- Default：主色填充，白字。
- Hover：亮度+6%。
- Focus：2px 主色外描边。
- Disabled：灰底灰字，不可点击。
- Loading：显示旋转占位，禁用点击。
- Error：用于危险操作（红底）。
- Empty：按钮不适用（由空状态卡片引导）。

### 2.2 Input / Select / Textarea
- Default：白底灰边。
- Hover：边框加深。
- Focus：主色描边+轻阴影。
- Disabled：灰底+禁用光标。
- Loading：下拉/联想加载骨架。
- Error：红色边框+错误文案。
- Empty：显示占位提示（如“请输入题干”）。

### 2.3 Table（题库/报表）
- Default：斑马纹可选，表头吸顶。
- Hover：行高亮。
- Focus：键盘导航行有描边。
- Disabled：批量操作按钮禁用。
- Loading：表格骨架行。
- Error：表格上方错误条。
- Empty：空态插画+“去创建”按钮。

### 2.4 Exam Card（考试卡片）
- Default：显示考试名、时长、状态、截止时间。
- Hover：卡片阴影增强。
- Focus：键盘可聚焦边框。
- Disabled：已结束/未开始状态不可进入。
- Loading：卡片骨架。
- Error：拉取失败提示重试。
- Empty：暂无待考，显示引导。

### 2.5 Answer Sheet（答题区）
- Default：题目区+答题卡+计时器。
- Hover：选项背景高亮。
- Focus：当前题锚点高亮。
- Disabled：交卷后全只读。
- Loading：切题骨架。
- Error：自动保存失败提示“重试”。
- Empty：题目加载为空时回退并告警。

### 2.6 Status Tag
- Default：草稿/已发布/进行中/已结束。
- Hover：不改变语义色，仅增加对比。
- Focus：可聚焦标签（筛选器内）。
- Disabled：不可切换筛选项。
- Loading：占位圆角块。
- Error：异常状态标签（红色）。
- Empty：无标签数据时显示“-”。

## 3. 页面草图说明
1. 登录页：双栏布局（品牌说明 + 登录表单），支持密码可见切换。
2. 考试列表页：筛选（状态/时间）+ 卡片网格。
3. 在线考试页：左侧题目正文，右侧答题卡与倒计时，顶部显示自动保存状态。
4. 成绩结果页：总分、通过状态、错题解析入口、证书下载。
5. 管理端题库页：表格+批量导入/导出。
6. 模板配置页：基础信息 + 组卷规则（固定+随机）分区。
7. 阅卷中心：主观题队列+评分抽屉。
8. 审计报表页：筛选区 + 指标卡 + 导出任务列表。

## 4. 样式变量建议
```css
:root {
  --color-primary: #2563EB;
  --color-secondary: #64748B;
  --color-success: #10B981;
  --color-warning: #F59E0B;
  --color-error: #EF4444;
  --color-info: #3B82F6;
  --color-bg: #F8FAFC;
  --color-surface: #FFFFFF;
  --color-border: #E2E8F0;
  --text-main: #0F172A;
  --text-sub: #475569;

  --radius-card: 8px;
  --radius-btn: 4px;
  --radius-input: 6px;

  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;

  --shadow-1: 0 1px 3px rgba(0,0,0,.1);
}
```
