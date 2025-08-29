# 初始化并行 git worktree 目录以支持并行 Claude Code 代理

## 变量

FEATURE_NAME: $ARGUMENTS
NUMBER_OF_PARALLEL_WORKTREES: $ARGUMENTS

## 执行以下命令

> 使用 Batch 和 Task 工具并行执行循环

- 创建新目录 `trees/`
- 对于 NUMBER_OF_PARALLEL_WORKTREES 中的每个 i
  - 执行 `git worktree add -b FEATURE_NAME-i ./trees/FEATURE_NAME-i`
  - 执行 `cd trees/FEATURE_NAME-i`, `git ls-files` 进行验证
- 执行 `git worktree list` 以验证所有目录树是否正确创建
