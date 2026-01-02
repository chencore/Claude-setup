---
allowed-tools: Read, Write, Bash
description: 从 UI 截图中提取设计系统并创建 design-system.json
argument-hint: [screenshot-paths...] - 要分析的 UI 截图路径
---

**创建设计系统 JSON**

要分析的 UI：$ARGUMENTS

**目标**
您需要从 `$ARGUMENTS` 中提供的截图中提取一个通用且可重用的设计系统，**不包括具体图像内容**，以便前端开发人员或 AI 代理可以将 JSON 作为构建一致 UI 的样式基础参考。

**任务**

1. 分析 `$ARGUMENTS` 中提供的截图以推断：
   - 调色板
   - 排版规则
   - 间距指南
   - 布局结构（网格、卡片、容器等）
   - UI 组件（按钮、输入框、表格等）
   - 边框半径、阴影和其他视觉样式模式
2. 创建一个 `design-system.json` 文件，明确定义这些规则，可以用来以一致的方式复制视觉语言。
3. 将 JSON 输出到 `prd` 文件夹，名称为：`design-system.json`

**约束**

- 不要从截图中提取具体内容（无文本、徽标、图标）。
- 仅专注于设计原则、结构和样式。
