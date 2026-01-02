---
description: 推送提交并创建/更新拉取请求，具有智能分支管理
argument-hint: [status] [base-branch] - status: 1=opened, 2=draft, 3=ready; base-branch 默认为 main
---
参数：$ARGUMENTS
## 参数

解析参数（灵活顺序）：
- **status**: `1`=opened, `2`=draft, `3`=ready（默认：新 PR=opened，更新=draft）
- **base-branch**: 目标分支（默认：`main`）

## 工作流

### 1. 起飞前检查

```bash
# 检查未提交的更改
git status --porcelain

# 检查远程跟踪
git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null

# 如果存在跟踪，与远程同步
git fetch origin
```

如果检测到未提交的更改，首先使用带有 subagent_type="smart-commit" 的 Task 工具提交更改。

### 2. 分支管理（关键）

**步骤 1：检测当前情况**
```bash
# 获取当前分支名称
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
echo "Current branch: $CURRENT_BRANCH"

# 检查是否在 main/master 上
if [[ "$CURRENT_BRANCH" == "main" || "$CURRENT_BRANCH" == "master" ]]; then
    echo "ON BASE BRANCH"
fi

# 计算未推送的提交数量
UNPUSHED=$(git rev-list origin/$CURRENT_BRANCH..HEAD --count 2>/dev/null || echo "0")
echo "Unpushed commits: $UNPUSHED"
```

**步骤 2：如果在 main/master 上且有未推送的提交 → 切出功能分支**

这很关键。如果您检测到：
- 当前分支是 `main` 或 `master`
- 有未推送的提交（UNPUSHED > 0）

那么您必须：
1. 分析提交以生成描述性的功能分支名称（例如 `feat/add-smart-commit-agents`）
2. 从当前 HEAD 创建功能分支
3. 切回 main 并将其重置到 origin
4. 切换到新功能分支

```bash
# 1. 获取提交用于命名
git log origin/main..HEAD --oneline

# 2. 创建功能分支（您根据提交生成名称）
git checkout -b feat/descriptive-name-here

# 3. 将 main 重置以匹配 origin（重要：保持 main 干净）
git checkout main
git reset --hard origin/main

# 4. 返回功能分支
git checkout feat/descriptive-name-here
```

**步骤 3：如果在功能分支上** → 跳过分支管理，继续 PR 状态。

### 3. PR 状态确定

```bash
# 检查当前分支是否有 PR
BRANCH=$(git rev-parse --abbrev-ref HEAD)
gh pr list --head "$BRANCH" --json number,state
```

- 如果用户提供了状态参数：使用它（1=opened，2=draft，3=ready）
- 如果没有状态：新 PR 默认为 "opened"，现有 PR 更新默认为 "draft"

### 4. 上下文收集

```bash
# 获取基础分支（默认：main）
BASE=${BASE_BRANCH:-main}

# 从基础分支到 HEAD 的所有提交
git log $BASE..HEAD --oneline

# 差异统计
git diff $BASE...HEAD --stat

# 完整差异用于分析
git diff $BASE...HEAD
```

### 5. 推送到远程

```bash
BRANCH=$(git rev-parse --abbrev-ref HEAD)

# 检查上游是否存在
if git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null; then
    git push
else
    git push -u origin "$BRANCH"
fi
```

### 6. PR 创建或更新

**对于新 PR**：

分析提交和更改以生成：
- 简洁、描述性的 PR 标题
- 包含摘要、提交列表、关键文件更改的综合描述

```bash
# 创建 PR（如果是草稿则添加 --draft）
gh pr create --title "title" --body "description" --base main

# 如果状态是 "ready"，创建后标记为就绪
gh pr ready
```

**对于现有 PR**：

```bash
# 获取 PR 编号
PR_NUM=$(gh pr list --head "$BRANCH" --json number -q '.[0].number')

# 添加评论列出新提交
gh pr comment $PR_NUM --body "New commits..."

# 如需更新 PR 状态
gh pr ready $PR_NUM --undo  # 转换为草稿
gh pr ready $PR_NUM         # 标记为就绪
```

## 关键约束

- 永远不要向 PR 内容添加 "Co-authored-by" 或 AI 签名
- 永远不要包含 "使用 Claude Code 生成" 或类似内容
- 永远不要在 PR 标题或描述中使用表情符号
- 仅使用现有的 git 用户配置
- 确保 PR 标题遵循项目约定（如果可用，分析最近的 PR）

## PR 描述格式

```markdown
## 摘要

[2-3 个要点描述内容和原因]

## 更改

- [关键更改 1]
- [关键更改 2]

## 提交

- `abc1234` - 提交消息 1
- `def5678` - 提交消息 2

## 更改的文件

[列出重要文件并简要说明]
```

## 边缘情况

1. **未配置远程**：报告错误，建议 `git remote add origin <url>`
2. **没有 gh CLI**：报告需要 GitHub CLI 进行 PR 操作
3. **权限问题**：报告并建议检查存储库访问权限
4. **分支落后于远程**：推送前先拉取并变基
5. **没有要推送的提交**：报告没有新提交

## 输出

清晰报告：
- 已推送分支：`<branch-name>`
- 已创建/更新 PR：`<PR-URL>`
- 状态：opened/draft/ready