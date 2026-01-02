---
allowedTools: Read, Write, Edit, Glob, Grep, WebFetch, TodoWrite, WebSearch, Task, Skill, SlashCommand, AskUserQuestion
description: 智能提示生成器，具有专业路由功能。直接创建通用提示；将 Claude Code 功能（插件、代理、技能、钩子、MCP、命令）路由到具有深度领域知识的专用工具。
argument-hint: [feature-type] [feature-name] - 可选：plugin、agent、skill、hook、mcp、command，或留空进行调查
---

# 提示生成器

智能提示生成器，具有专用工具路由功能。将 Claude Code 功能（插件、代理、技能、钩子、MCP、命令）路由到具有深度领域知识的专用工具。直接处理通用 AI 提示并进行元评估。

## 上下文工程原则

上下文工程是在 LLM 推理过程中优化令牌集合。每个生成的提示必须体现这些原则：

**上下文是有限的**
- LLM 有有限的注意力预算；随着上下文增长，性能会下降
- 每个令牌都会消耗容量——将上下文视为宝贵资源
- 目标：以尽可能小的、高信噪比令牌集合来最大化结果

**优化信噪比**
- 使用清晰、直接的语言而非冗长的解释
- 无情地删除冗余信息
- 专注于驱动行为的高价值令牌

**渐进式发现**
- 使用轻量级标识符而非完整的数据转储
- 在需要时动态加载详细信息
- 即时信息 > 预加载的上下文

**写作风格**

| 模式 | ✅ 好 | ❌ 坏 |
|---------|---------|--------|
| 清晰胜过完整 | "处理前验证输入" | "你应该总是确保验证..." |
| 直接 | "使用 calculate_tax 工具和金额、司法管辖区" | "你可能想考虑使用..." |
| 祈使语气 | "分析响应" | "我将分析响应" |
| 结构化约束 | 要求的带项目符号列表 | 包含要求的长段落 |

## 执行流程

### 第 1 阶段：上下文发现

**如果提供了参数 ($ARGUMENTS)：**
解析功能类型和可选名称。识别的类型：
- **路由**：plugin、agent、skill、hook、mcp、command（重定向到专用工具）
- **直接**：通用 AI 提示（由 promptcraft 处理）

**如果没有参数：**
根据用户的请求确定这是：
- Claude Code 功能（plugin、agent、skill、hook、mcp、command）
- 通用 AI 提示

**对于 Claude Code 功能：**
首先始终 fetch claude /docs <feature-type> 以加载当前文档和最佳实践。

**始终使用 AskUserQuestion 工具提出 3-5 个针对性问题** 来了解：
- 主要目标和成功标准
- 目标用例和约束
- 预期的输入/输出
- 复杂性级别和范围
- 工具要求（如果适用）

### 第 1.5 阶段：专用工具路由

识别功能类型后，检查是否有专用工具可以提供更好的结果。将复杂功能路由到专用工具；较简单的功能继续使用 promptcraft。

**路由决策表：**

| 功能类型 | 路由到 | 方法 | 原因 |
|--------------|----------|--------|--------|
| **Plugin** | `/plugin-dev:create-plugin` | SlashCommand | 包含验证的完整 8 阶段工作流 |
| **Agent** | `plugin-dev:agent-creator` agent | Task | Claude Code 的生成模式，带有 `<example>` 块 |
| **Skill** | `plugin-dev:skill-development` skill | Skill | 渐进式揭示，第三人称描述 |
| **Command** | `plugin-dev:command-development` skill | Skill | frontmatter、参数、bash 执行模式 |
| **Complex Hook** | `plugin-dev:hook-development` skill | Skill | 深度 API 知识，9 个事件，安全模式 |
| **Simple Hook** | `/hookify:hookify` | SlashCommand | 快速行为规则，无需编码 |
| **MCP Server** | `plugin-dev:mcp-integration` skill | Skill | 服务器类型、身份验证、.mcp.json 配置 |
| **General Prompt** | 继续在这里 | — | promptcraft 核心用例 |

**路由逻辑：**

| 功能 | 操作 | 工具调用 |
|---------|--------|-----------|
| Plugin | 始终重定向 | `SlashCommand: /plugin-dev:create-plugin [description]` |
| Agent | 始终重定向 | `Task: subagent_type=plugin-dev:agent-creator` |
| Skill | 始终重定向 | `Skill: plugin-dev:skill-development` |
| Command | 始终重定向 | `Skill: plugin-dev:command-development` |
| MCP Server | 始终重定向 | `Skill: plugin-dev:mcp-integration` |
| Hook | 首先询问复杂性 | 见下文 |
| General Prompt | 继续到第 2 阶段 | — |

**对于 Hooks**：使用 AskUserQuestion 确定复杂性：
- "简单的警告/阻止规则" → `SlashCommand: /hookify:hookify [description]`
- "带有验证的复杂钩子" → `Skill: plugin-dev:hook-development`
- "我不知道" → 建议 `/hookify:hookify`（分析会话中的模式）

**路由后：**
- 如果重定向：让专用工具完成任务，然后继续 **第 4 阶段** 进行优化审查
- 如果继续（通用提示）：继续到第 2 阶段

### 第 2 阶段：提示架构（仅通用提示）

**如果在第 1.5 阶段路由则跳过。** 对于通用 AI 提示，分析需求：
- 识别核心目标
- 确定复杂性（简单/中等/复杂）
- 将所需能力映射到可用工具
- 结构化适当的格式

**工具智能**（针对 Claude Code 功能）：
基于提示要求，建议工具：
- **Bash**：如果需要系统命令、测试或 git 操作
- **Read/Write/Edit**：用于文件操作
- **Grep/Glob**：用于代码搜索和导航
- **WebSearch/WebFetch**：用于研究或文档查找
- **Task**：用于委派给子代理
- SlashCommand、AskUserQuestion 等。
- 对于功能，慷慨地授予工具访问权限，无需保守

显示工具选择及其理由：
"选择的工具：Read、Grep、Bash(git:*)、Bash(pytest:*)、AskUserQuestion
理由：代码分析需要文件阅读和搜索；测试需要 git 和 pytest 命令。
要修改此选择吗？[显示排除的内容及原因]"

**格式适配**：
- **简单提示**：直接指令，最小结构，无章节
- **中等提示**：轻量级组织，在需要清晰性的地方使用 ## 标题
- **复杂提示**：结构化章节，清晰的层次结构，过程步骤

### 第 3 阶段：提示生成（仅通用提示）

**如果在第 1.5 阶段路由则跳过。** 对于通用 AI 提示：

**语义章节结构**

使用这些语义章节组织提示正文（只包含需要的部分）：

| 章节 | 目的 | 何时包含 |
|---------|---------|-----------------|
| **Background** | 最小基本上下文 | 仅当任务需要领域知识时 |
| **Instructions** | 核心指令，祈使语气 | 总是 |
| **Examples** | 展示而非讲述，简洁，代表性 | 当行为需要演示时 |
| **Constraints** | 边界、限制、成功标准 | 当存在歧义时 |

**格式模板：**

**Commands**（frontmatter + body）：
```markdown
---
allowedTools: <根据第 4.5 阶段智能选择>
description: <简洁—用户触发，无需自动触发>
argument-hint: [arg] - 描述
---

<根据需要包含语义章节>
```

**Agents**（frontmatter + body）：
```markdown
---
name: <agent-name>
description: <触发条件和变化短语 + 能力>
tools: <根据第 4.5 阶段的工具名称>
model: sonnet
---

<根据需要包含语义章节>
```

**Skills**（SKILL.md + 可选 references/）：
```markdown
---
description: <丰富的触发条件和变化示例—"当用户要求 X 时使用"，"Y"，"Z"，或提及"A"，"B">
---

<渐进式揭示：核心指令优先，细节在后>
```

**Generic Prompts**：无 frontmatter。纯指令正文。

**构建规则**：
- 在第一句中明确说明目标
- 使用祈使语气（"分析"、"生成"、"识别"）
- 无第一人称叙述（"我将"、"我是"）
- 仅在理解需要时提供上下文
- 仅在复杂结构化数据时使用 XML 标签
- 仅在澄清期望时提供示例
- 每个词都必须有其存在价值

### 第 4 阶段：功能优化（所有 Claude Code 功能）

**在重定向的工作流完成后或第 3 阶段后应用于通用提示。** 此阶段确保渐进式揭示和 frontmatter 优化。

#### Frontmatter 优先级

Frontmatter **首先获得模型注意力**—无情地优化。不同功能需要不同的策略：

| 功能 | 描述策略 | 触发设计 |
|---------|---------------------|----------------|
| **Command** | 简洁、以行动为中心 | 用户触发（无自动触发） |
| **Skill** | 丰富的触发条件和变化短语 | 高度期望自动触发 |
| **Agent** | 清晰的条件 + 能力 | 中度自动触发 |
| **Hook** | 事件特定的清晰性 | 事件驱动 |

**Command Frontmatter**：
- `description`：简洁的行动摘要（少于 60 个字符）—用户在 `/help` 中看到
- `allowedTools`：适当限制范围（见下文）
- `argument-hint`：清晰记录预期参数
- Commands 是 Claude 的指令，不是给用户的消息

**Skill/Agent 触发优化**：
- 包含变化的触发短语，而非精确的关键词
  - 好："创建一个代理"，"构建一个新代理"，"为我创建一个代理..."，"用于...的代理"
  - 坏：只有"create agent"
- 平衡令牌成本与触发准确性
- 更多示例 = 更高的自动触发概率
- 使用 AskUserQuestion："这个应该自动触发还是明确触发的频率如何？"

#### allowedTools 策略

**默认宽松，仅限制高风险操作。**

| 工具 | 默认 | 何时限制 |
|------|---------|---------------|
| Read, Glob, Grep | 总是允许 | 从不 |
| Edit, Write | 允许 | 写入系统路径时 |
| WebSearch, WebFetch | 允许 | 仅限离线功能时 |
| Task | 允许 | 沙盒功能时 |
| **Bash** | 使用模式 | 始终限制：`Bash(git:*)`，`Bash(npm:*)`，`Bash(pytest:*)` |
| **KillShell** | 省略 | 仅在明确需要时包含 |
| AskUserQuestion | **必需** | 当进行访谈/确认时从不省略 |
| SlashCommand | **必需** | 当委派给工作流时从不省略 |
| Skill | 允许 | 当存在委派机会时 |

**必需包含：**
- 如果提示有任何用户交互/确认 → 包含 `AskUserQuestion`
- 如果提示委派给其他功能 → 包含 `SlashCommand` 和/或 `Skill`
- 明确指导何时使用："使用 AskUserQuestion 工具确认..." 或 "使用 SlashCommand: /commit"

#### 委派和模块化

在最终确定前，扫描委派机会：

```
查看可用的：skills、commands、agents、MCPs
对于每个工作流步骤，问："我们已经有了这个吗？"
```

**常见委派模式：**
- Git 提交 → `SlashCommand: /commit`
- 代码审查 → `SlashCommand: /code-review`
- 插件创建 → `SlashCommand: /plugin-dev:create-plugin`
- 钩子创建 → `Skill: plugin-dev:hook-development`
- 命令创建 → `Skill: plugin-dev:command-development`
- 文档查找 → `SlashCommand: /docs [topic]`

**始终使用完整限定名称：**
- `Skill: plugin-dev:hook-development`（不仅仅是"hook-development"）
- `SlashCommand: /plugin-dev:create-plugin`（不仅仅是"create-plugin"）
- `Task: subagent_type=plugin-dev:agent-creator`

#### 正文审查清单

| 问题 | 修复 |
|-------|-----|
| 冗长的代码块 | 提供模式，提供适应边缘情况的一般指令 |
| 精确的关键词匹配 | 替换为基于意图的语言（Claude 推断） |
| 硬编码的路径/值 | 使用变量或让 Claude 推断 |
| 冗余示例 | 保留 2-3 个代表性案例，移除相似的 |
| 过度指定的过程 | 相信 Claude 的智能—提供指导而非指令 |

**关键原则**：Claude 很聪明且善于推断—精确引导，不要过度指定。

### 第 5 阶段：元评估

**评估生成/优化的提示：**

| 维度 | 标准 |
|-----------|----------|
| **清晰度 (0-10)** | 指令清晰明确，目标清楚 |
| **精确度 (0-10)** | 适当的特异性，不过度约束 |
| **效率 (0-10)** | 令牌经济性—每个令牌的最大价值 |
| **完整性 (0-10)** | 涵盖需求，无缺口或过剩 |
| **可用性 (0-10)** | 实用、可操作，适合目标用途 |

**目标：9.0/10.0**

显示评估，然后：
- 如果 < 9.0：解决弱点进行改进，重新评估一次
- 如果 ≥ 9.0：继续交付

### 第 6 阶段：交付

**确定输出位置：**
- Commands → `~/.claude/commands/<name>.md`
- Agents → `~/.claude/agents/<name>.md`
- Skills → `~/.claude/skills/<name>/SKILL.md`
- Hooks → `~/.claude/hooks/<name>.md`
- Generic/unspecified → `./<name>_prompt.md`（项目根目录）
- 用户指定的路径 → 按提供的位置

**写入前：**
"将提示写入：[路径]
这将 [创建新/覆盖现有] 文件。
继续吗？"

**写入后：**
确认创建并提供使用说明（如果适用，例如命令使用 `/command-name` 调用）。

## 质量标准

应用上下文工程原则（见上文）。另外：

**格式经济性：**
- 简单任务 → 直接指令，无章节
- 中等任务 → 轻量级组织，带有标题
- 复杂任务 → 完整语义结构

**平衡灵活性与精确性：**
- 足够宽松以支持创造性探索
- 足够紧密以避免歧义

**无情地删除**：填充短语、明显含义、冗余框架、过度礼貌

## 错误处理

| 问题 | 操作 |
|-------|--------|
| 缺少上下文 | 请求澄清 |
| 不明确的需求 | 要求示例或约束 |
| 工具冲突 | 解释限制，建议替代方案 |
| 路径问题 | 验证目录，在确认下创建 |

## 成功标准

生成的提示必须：在使用时实现目标，在元评估中得分 ≥9.0/10，无需修改即可立即使用。

顺序执行阶段。完成每个阶段后再继续。