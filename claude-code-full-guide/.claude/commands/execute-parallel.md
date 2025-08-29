# Parallel Task Version Execution

## 变量

FEATURE_NAME: $ARGUMENTS
PLAN_TO_EXECUTE: $ARGUMENTS
NUMBER_OF_PARALLEL_WORKTREES: $ARGUMENTS

## 指令

我们将创建 NUMBER_OF_PARALLEL_WORKTREES 个新的子代理，这些子代理使用 Task 工具并行创建同一功能的 N 个版本。

务必阅读 PLAN_TO_EXECUTE。

这使我们能够并行地在同一功能上进行构建，以便在隔离环境中测试和验证每个子代理的更改，然后选择最佳的更改。

第一个代理将在 trees/<FEATURE_NAME>-1/ 中运行。
第二个代理将在 trees/<FEATURE_NAME>-2/ 中运行。
...
最后一个代理将在 trees/<FEATURE_NAME>-<NUMBER_OF_PARALLEL_WORKTREES>/ 中运行。

trees/<FEATURE_NAME>-`<i>`/ 中的代码将与当前分支中的代码相同。它将被设置并准备好供您端到端地构建该功能。

每个代理将在各自的工作空间中独立实现 PLAN_TO_EXECUTE 中详细说明的工程计划。

当子代理完成工作后，让子代理在其各自工作空间的根目录中生成一个全面的 `RESULTS.md` 文件，报告最终的更改。

确保代理不运行任何测试或其他代码——仅专注于代码更改。
