# Claude-setup 🚀

一个高度定制化的 Claude Code CLI 工作流增强配置，通过模块化架构为开发者提供全方位的 AI 辅助支持。

## 项目简介

Claude-setup 是专为 Claude Code CLI 设计的增强型配置系统，通过精心设计的模块化架构，将 AI 辅助编程提升到一个新的水平。项目集成了智能代理、实用命令、可重用技能和丰富的插件系统，帮助开发者从编码架构到个人效率的各个方面获得 AI 支持。

## 核心特性

### 智能代理系统

项目包含 7 个专业化的领域特定 AI 代理，每个代理都专注于特定的技术领域：

- **architecture-auditor** - 软件架构专家，负责 SOLID 原则审查、设计模式分析、系统架构评估、技术债务评估和架构反模式识别
- **branch-cleaner** - Git 分支管理专家
- **code-auditor** - 代码质量审查专家
- **performance-auditor** - 性能优化专家
- **security-auditor** - 安全审计专家
- **smart-commit** - 智能提交生成专家
- **test-engineer** - 测试工程专家

### 实用命令集合

包含 26 个精心设计的实用命令，涵盖开发工作的各个方面：

- **brainstorm** - 头脑风暴助手，帮助快速生成创意和解决方案
- **commit** - 智能提交助手，自动生成符合规范的提交信息
- **code-review** - 代码审查工具，全面检查代码质量
- **debug-error** - 错误调试助手，快速定位和解决问题
- **interview** - 深度访谈工具，探索想法和需求
- **make-it-pretty** - 代码美化工具，提升代码可读性
- **parallel-agents** - 并行代理执行，提高工作效率
- **performance-audit** - 性能审计工具
- **security-audit** - 安全审计工具
- **tech-debt** - 技术债务管理
- **understand** - 代码理解助手

### 可重用技能模块

提供 8 个高级技能模块，可在不同场景中重复使用：

- **session-search** - 会话搜索技能，支持通过关键字、时间范围搜索历史对话，自动提取学习成果和知识缺口
- **gemini-imagegen** - Gemini 图像生成，支持多轮对话编辑和图像合成
- **learn-anything** - 通用学习系统
- **modern-frontend-design** - 现代前端设计工具
- **tapestry** - 复杂系统分析工具
- **youtube-transcript** - YouTube 转录服务
- **article-extractor** - 文章提取工具
- **content-research-writer** - 内容研究与写作助手
- **ship-learn-next** - Next.js 学习与部署工具

### 增强状态栏

自定义的多功能状态栏，提供丰富的信息展示：

- 实时上下文窗口使用率监控
- 天气显示（带缓存机制）
- IST 时间显示
- 季度进度追踪
- 生命周期百分比计算
- Spotify 音乐信息集成
- Git 状态显示
- MCP 服务状态监控

### 输出样式定制

提供多种专业的输出样式配置：

- **technical-architect** - 技术架构师风格
- **prd-writer** - 产品需求文档编写者
- **legacy-explorer** - 遗留代码探索者

### 事件钩子系统

包含 5 个强大的钩子脚本，在特定事件时自动执行：

- **session-context-loader** - 会话上下文自动加载
- **learning-style-session-start** - 学习风格会话启动
- **parse-session** - 会话解析器
- **security_reminder_hook** - 安全提醒钩子
- **type_check** - 类型检查工具

## 项目结构

```
Claude-setup/
├── agents/                      # 智能代理模块
│   ├── architecture-auditor.md
│   ├── branch-cleaner.md
│   ├── code-auditor.md
│   ├── performance-auditor.md
│   ├── security-auditor.md
│   ├── smart-commit.md
│   └── test-engineer.md
├── commands/                    # 命令模块
│   ├── brainstorm.md
│   ├── commit.md
│   ├── code-review.md
│   ├── debug-error.md
│   ├── interview.md
│   └── ... (26个命令)
├── skills/                      # 技能模块
│   ├── session-search/
│   ├── gemini-imagegen/
│   ├── learn-anything/
│   └── ... (8个技能)
├── hooks/                       # 钩子脚本
│   ├── session-context-loader.sh
│   ├── learning-style-session-start.sh
│   ├── parse-session.py
│   ├── security_reminder_hook.py
│   └── type_check.py
├── output-styles/              # 输出样式配置
│   ├── technical-architect.md
│   ├── prd-writer.md
│   └── legacy-explorer.md
├── plugins/                    # 插件管理
├── settings.json               # 主配置文件
├── mcp-config.template.json    # MCP 服务配置模板
├── statusline-script.sh        # 状态栏脚本
├── CLAUDE.md                   # 项目核心工作流文档
└── README.md                   # 项目说明文档
```

## 配置说明

### settings.json

核心配置文件，包含以下重要设置：

- **permissions** - 文件访问权限管理
- **model** - 使用的 AI 模型（glm-4.7）
- **hooks** - 会话开始和停止时的钩子配置
- **statusLine** - 自定义状态栏设置
- **enabledPlugins** - 插件启用/禁用配置
- **forceLoginMethod** - 强制使用 claudeai 登录
- **syntaxHighlightingDisabled** - 禁用语法高亮
- **alwaysThinkingEnabled** - 启用思考模式

### MCP 服务集成

支持多个 Model Context Protocol (MCP) 服务器的集成：

- **playwright** - 网页自动化
- **brave-search** - 搜索引擎
- **hf-mcp-server** - Hugging Face 模型服务
- **exa** - 高级搜索引擎
- **notion** - Notion 集成
- **firecrawl** - 网页抓取

## 工作流特性

### 核心工作流

项目定义了一套完整的核心工作流程：

- **工具使用与效率** - 并行工具调用、智能上下文管理
- **调查与研究** - 深入代码理解、系统化信息搜索
- **默认保持谨慎** - 明确用户意图后再执行操作

### 代码质量标准

- **高质量解决方案** - 使用标准工具，避免硬编码和捷径
- **避免过度工程** - 保持简单专注，只做必要更改
- **健壮性** - 提供可维护和可扩展的解决方案

### 前端美学指导

- **独特字体选择** - 避免通用字体，使用独特的字体组合
- **连贯配色** - 使用 CSS 变量保持一致性
- **动画效果** - 精心编排的页面加载和微交互
- **避免 AI 糟粕** - 创造有创意、独特的前端设计

### 沟通风格

- **流畅散文** - 使用清晰、流畅的段落和句子
- **适度格式化** - 主要使用 markdown 进行代码和标题格式化
- **自然引导** - 将信息自然融入句子，避免过度列表

## 安装与使用

### 环境要求

- Claude Code CLI 工具
- Git（用于版本控制）
- Bash 支持（用于钩子脚本）

### 配置步骤

1. 克隆项目到本地配置目录
2. 根据 `settings.json` 模板配置你的设置
3. 如需使用 MCP 服务，配置 `mcp-config.template.json`
4. 确保 `statusline-script.sh` 有执行权限
5. 配置钩子脚本的执行权限

### 自定义配置

根据个人需求，你可以：

- 启用或禁用特定插件
- 自定义状态栏显示内容
- 添加新的命令或技能
- 调整钩子脚本行为
- 修改输出样式

## 最新更新

- **7849400** - 翻译为中文版本
- **621eeae** - 更新市场时间戳
- **7c8291f** - 添加深度访谈命令
- **e2f33d3** - 优化会话搜索功能
- **89fcb63** - 简化上下文百分比计算

## 技术栈

- **AI 模型**: GLM-4.7
- **脚本语言**: Bash, Python
- **配置格式**: JSON, Markdown
- **版本控制**: Git
- **协议**: MCP (Model Context Protocol)

## 项目亮点

### 模块化架构

清晰分离的功能模块，易于扩展和维护。每个模块都可以独立使用或组合使用，提供最大的灵活性。

### 智能上下文感知

自动跟踪项目状态、Git 变更、会话历史，提供上下文感知的 AI 辅助。

### 持续学习系统

自动从对话中提取学习成果和知识缺口，帮助开发者持续成长。

### 多维度监控

从代码质量到个人效率的全方位追踪，包括技术债务、性能指标和安全状况。

### 中文优化

专门针对中文用户优化的配置和文档，提供更好的本地化体验。

## 贡献指南

欢迎贡献新的命令、技能或代理！在提交 PR 之前，请确保：

- 遵循现有的代码风格和约定
- 提供清晰的文档说明
- 测试你的更改
- 更新相关文档

## 许可证

本项目采用开源许可证。具体条款请查看 LICENSE 文件。

## 联系方式

如有问题或建议，欢迎通过以下方式联系：

- 提交 Issue
- 发起 Pull Request
- 参与项目讨论

---

让 AI 成为你的编程伙伴，提升开发效率，创造更优秀的代码！
