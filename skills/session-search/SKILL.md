---
name: session-search
description: 通过关键字、时间范围或 ID 搜索过去的对话。在"搜索会话"、"找到那场对话"、"我们做了什么"、"查找关于 X 的过去讨论"、"我们在哪"、"我学了什么"、"知识缺口"、"回顾"时触发。也在以下情况触发："我们讨论了..."、"你提到过..."、指代先前工作的过去时动词、没有上下文的物主代词（"我的项目"、"我的认证系统"）、引用未共享上下文的假设性问题。提取干净的对话数据并应用分析透镜以获得结构化见解。
---

# session-search

## 提取

```bash
python3 ~/.claude/skills/session-search/scripts/extract_conversations.py --days 3
```

| 选项 | 效果 |
|------|------|
| `--days N` | 从最后活动时间算起的天数（不包括今天） |
| `--from-today` | 从今天算起的天数 |
| `--all-projects` | 跨项目（隐含 --from-today） |
| `--project /path` | 过滤到特定项目 |
| `--compact` | 无元数据，更多对话内容 |
| `--min-exchanges N` | 跳过交换次数 < N 的会话 |
| `--ids abc,def` | 获取特定对话 |
| `--paths /path.jsonl` | 直接文件路径 |

**输出**：默认显示文件、工具、错误 + 对话。`--compact` 省略元数据。

**时间模式**：`--days N` 从最后活动时间开始计数（返回旧项目时有用）。`--all-projects` 或 `--from-today` 从今天开始计数（基于日历）。

## 工作流程

1. **Grep 搜索关键字**（生成语义同义词）：
   ```bash
   grep -l -E "auth|login|session" ~/.claude/projects/*/*.jsonl
   ```

2. **提取**：`python3 ... --paths /found/conv.jsonl`

3. **应用透镜**：使用下面的参数和问题。

---

## 透镜

### 路由

| 用户说 | 透镜 |
|--------|------|
| "我们在哪"、"回顾" | restore-context |
| "我学到了什么"、"反思" | extract-learnings |
| "缺口"、"困难" | find-gaps |
| "导师"、"回顾流程" | review-process |
| "回顾"、"项目回顾" | run-retro |
| "决定"、"CLAUDE.md" | extract-decisions |
| "坏习惯"、"反模式" | find-antipatterns |

### 参数

| 透镜 | 天数 | 标志 | 同时收集 |
|------|------|------|----------|
| restore-context | 3 | — | `git status`, `git log -10` |
| extract-learnings | 14 | `--all-projects --compact` | — |
| find-gaps | 30 | `--all-projects --compact` | — |
| review-process | 14 | `--all-projects --compact` | 最近的 git 日志 |
| run-retro | 30 | `--project /path` | 完整的 git 历史 |
| extract-decisions | 90 | `--project /path` | — |
| find-antipatterns | 30 | `--all-projects --compact` | — |

`--min-exchanges 2` 或 `3` 过滤掉短会话并减少噪音。

### 核心问题

| 透镜 | 询问 |
|------|------|
| restore-context | 有什么未完成？下一步是什么？ |
| extract-learnings | 理解在何处转变？什么错误变成了教训？ |
| find-gaps | 什么主题反复出现？在哪里需要持续指导？ |
| review-process | 编码前有规划吗？调试是系统性的吗？ |
| run-retro | 解决方案是如何演变的？什么有效？什么令人痛苦？ |
| extract-decisions | 讨论了哪些权衡？被拒绝的原因是什么？ |
| find-antipatterns | 什么错误重复出现？什么困惑持续存在？ |

**后续**：find-gaps → 建议 `learn-anything`。extract-decisions → 建议 `/updateclaudemd`。

### Grep 信号

使用这些模式在提取前查找相关会话：

| 透镜 | Grep 模式 |
|------|----------|
| extract-learnings | `learned\|realized\|understand now\|clicked\|got it` |
| find-gaps | `confused\|don't understand\|struggling\|help with` |
| extract-decisions | `decided\|chose\|instead of\|trade-off\|because` |
| find-antipatterns | `again\|same mistake\|repeated\|forgot` |

---

## 综合

### 原则

1. **优先重要事项** — 3-5 个关键发现，而不是详尽的列表
2. **具体化** — 文件路径、日期、项目名称
3. **可操作** — 每个发现都建议一个响应
4. **展示证据** — 引用或参考
5. **保持可扫描** — 清晰的结构，没有大段文字

### 结构

```markdown
## [分析类型]: [范围]

### 总结
[2-3 句话]

### 发现
[按适合的方式组织：类别、时间线、严重程度]

### 模式
[跨观察结果]

### 建议
[可操作的下一步]
```

### 长度

默认：300-500 字。仅在数据 warrant 时扩展。