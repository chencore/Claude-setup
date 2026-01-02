---
allowed-tools: Bash, Task, Read, TodoWrite, Glob
description: 使用 git worktrees 和多个代理并行执行任务
argument-hint: [task-description] - 要并行执行的任务
---

# 并行任务执行

参数：$ARGUMENTS

## 步骤 1：设置 git worktree

首先，根据所需的并行代理数量，在 '.trees' 文件夹中设置几个 git worktree，这样我们就可以为实验提供不同的沙盒环境
运行 `git worktree add .trees/<branch-name>`
将 branch-name 替换为反映意义的良好名称
之后，对于每个分支，我们应该进入该文件夹（使用绝对路径）并进行依赖安装来设置。始终首先确定包管理器、虚拟环境可用性详细信息。

## 步骤 2：启动并行子代理

我们将创建 N 个新的子代理，使用 Task 工具在每个 git worktree 中并行执行。
这使我们能够并行地并发构建功能，这样我们就可以隔离地测试和验证每个子代理的更改，然后选择最佳更改。
第一个代理将在 trees/<branch-name-1>/ 中运行
第二个代理将在 trees/<branch-name-2>/ 中运行
...
最后一个代理将在 trees/<branch-name-n>/ 中运行

每个树中的代码将与当前分支中的代码相同。它将被设置好，准备让您从头到尾构建功能。
每个代理将独立地在各自的工作空间中基于任务实施工程计划。
当子代理完成工作时，让子代理在各自工作空间的根目录中创建一个全面的 `RESULTS.md` 文件来报告其最终更改。
确保代理不运行 start.sh 或任何其他会启动服务器或客户端的脚本 - 仅专注于代码更改。
