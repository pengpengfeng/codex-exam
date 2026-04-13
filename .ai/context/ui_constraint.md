# UI 约束配置

## 品牌色

- 主色（Primary）：#2563EB（企业蓝）
- 辅助色（Secondary）：#64748B（中性灰）
- 成功色（Success）：#10B981（绿色）
- 警告色（Warning）：#F59E0B（橙色）
- 错误色（Error）：#EF4444（红色）
- 信息色（Info）：#3B82F6（蓝色）

## 风格调性

- **整体风格**：专业、简洁、高效（B2B企业内部系统）
- **设计语言**：Material Design 3.0 简化版
- **圆角**：8px（卡片）、4px（按钮）、6px（输入框）
- **阴影**：0 1px 3px rgba(0,0,0,0.1)
- **边框**：1px solid #E2E8F0

## UI组件库

- **不使用第三方UI组件库**（需自行设计组件）
- 使用原生CSS + CSS Variables
- 图标：Heroicons（SVG格式，内联）

## 字体

- 中文字体：-apple-system, "PingFang SC", "Microsoft YaHei", sans-serif
- 英文/数字：Inter, -apple-system, BlinkMacSystemFont, sans-serif
- 代码：JetBrains Mono, "Cascadia Code", monospace

## 间距系统

- 基础单位：4px
- 常用间距：4px, 8px, 12px, 16px, 24px, 32px, 48px, 64px
- 卡片内边距：24px
- 列表项间距：12px
- 表单字段间距：16px

## 断点（响应式）

- 移动端：< 768px
- 平板：768px - 1024px
- 桌面：> 1024px

## 动画

- 过渡时长：200ms（快速）、300ms（标准）
- 缓动函数：ease-out
- 禁用复杂动画（MVP阶段）

## 无障碍

- 遵循WCAG 2.1 AA级标准
- 焦点可见：outline: 2px solid #2563EB
- 最小触控目标：44px × 44px
- 颜色对比度：至少4.5:1

## 组件状态

所有交互组件必须定义以下状态：

- 默认（Default）
- 悬停（Hover）
- 聚焦（Focus）
- 禁用（Disabled）
- 加载（Loading）
- 错误（Error）
- 空状态（Empty）
