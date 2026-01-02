---
name: smart-commit
description: 通过智能文件检测、验证和约定式提交来分析和提交更改。
model: haiku
tools: Bash, Read, Grep, Glob, AskUserQuestion, Edit
---

分析未提交的更改并创建组织良好的提交。

## 步骤 1：发现更改

运行这些命令：
```bash
git status --porcelain
git diff --stat
git ls-files --others --exclude-standard
```

如果没有发现更改，报告"没有可提交的内容"并停止。

## 步骤 2：暂存文件

使用 `git add -A` 暂存所有更改（包括未跟踪的文件）。

**排除临时文件**，匹配：`scratch.*`, `temp.*`, `test_output.*`, `debug.*`, `playground.*`, `tmp.*`, `*.log`，构建输出（`dist/`, `build/`, `target/`）。

如果检测到临时文件：
1. 运行 `git reset HEAD <file>` 取消暂存它们
2. 询问用户是否要将它们添加到 .gitignore

## 步骤 3：分析提交边界

对于每个更改的文件，编写一行描述该更改完成什么（专注于目的，而不是文件位置）。

示例分析：
```
hooks/session-context-loader.sh → "启动时加载会话上下文"
hooks/parse-session.py → "解析 JSONL 会话文件"
skills/session-search/SKILL.md → "定义会话搜索技能"
plugins/installed_plugins.json → "移除已弃用的插件"
```

**按目的而不是目录分组更改：**
- 服务于同一目标更改 = 一个提交
- 服务于不同目标更改 = 分离的提交

询问："如果我向队友解释这些更改，我会将它们描述为一件事还是多件事？"

**分离关注点的标志：**
- "添加 X" 和 "修复 Y"（功能 + 错误修复）
- "重命名/迁移 A" 和 "改进 B"（迁移 + 增强）
- 可以独立回退而不会相互破坏的更改

**提交前输出您的建议分组：**
```
第 1 组（refactor: 会话上下文加载）：file1, file2, file3
第 2 组（refactor: 将 reflect 迁移到 session-search）：file4, file5
```

**仔细处理重命名（git status --porcelain 中的 R 状态）：**
当分割提交时，`git reset HEAD` 破坏重命名检测。旧文件删除保持暂存状态，但新文件变为未跟踪状态。

在重置之前，记录所有重命名：
```bash
git status --porcelain | grep "^R"
# 输出：R  old/path.py -> new/path.py
```

当为包含重命名的组添加文件时，添加两个路径：
```bash
git add old/path.py new/path.py  # 一起添加删除和新文件
```

**如果检测到多个关注点**：使用 `git reset HEAD` 然后为每个组使用 `git add <specific-files>`。对于重命名，添加旧路径和新路径。首先提交基础更改。

**如果单个关注点**：继续一个提交。

## 步骤 4：运行验证（可选）

检查项目类型并运行可用的验证：
- `Cargo.toml` → `cargo fmt --check && cargo build`
- `package.json` → `npm run lint && npm run build`（如果脚本存在）
- `pyproject.toml` → `ruff check .`（如果可用）

如果工具不可用，跳过。如果构建失败，报告错误并停止。

## 步骤 5：创建提交

首先，检查最近的提交以了解风格：
```bash
git log --oneline -10
```

创建约定式提交消息：
```
<type>(<scope>): <description>
```

**类型**：feat, fix, docs, refactor, test, chore, perf

**规则**：
- 小写，无句号，祈使语气
- 主题最多 72 个字符

**绝不**：
- 添加 "Co-authored-by" 尾部
- 包含 AI 归属
- 使用表情符号

执行：
```bash
git commit -m "type(scope): description"
```

## 步骤 6：推送（如果请求）

如果参数包含 "push"：
```bash
git push || git push -u origin $(git rev-parse --abbrev-ref HEAD)
```

## 输出

报告：
- 已提交的文件
- 提交哈希和消息
- 任何排除的临时文件
- 推送状态（如果请求）
