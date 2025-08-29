# Claude Code Hooks 示例

本目录包含Claude Code的示例钩子，展示了如何为您的AI编码工作流程添加确定性行为。

## 什么是钩子？

钩子是用户定义的shell命令，它们在Claude Code的生命周期中特定点执行。它们控制Claude Code的行为，确保某些操作始终执行，而不是依赖AI选择运行它们。

## 本目录中的文件

1. **format-after-edit.sh** - 一个PostToolUse钩子，在文件编辑后自动格式化代码
2. **example-hook-config.json** - 示例配置，展示如何设置各种钩子

## 如何使用这些钩子

### 选项1：复制到您的设置文件

将`example-hook-config.json`中的钩子配置复制到您的Claude Code设置：

**项目级** (`.claude/settings.json`):

```bash
# 如果不存在则创建设置文件
touch .claude/settings.json

# 从example-hook-config.json添加钩子配置
```

**用户级** (`~/.claude/settings.json`):

```bash
# 将钩子应用到所有Claude Code会话
cp example-hook-config.json ~/.claude/settings.json
```

### 选项2：使用单个钩子

1. 将钩子脚本复制到项目中：

```bash
cp format-after-edit.sh /your/project/.claude/hooks/
chmod +x /your/project/.claude/hooks/format-after-edit.sh
```

2. 添加到settings.json：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/format-after-edit.sh"
          }
        ]
      }
    ]
  }
}
```

## 可用钩子事件

- **PreToolUse**: 工具执行前（可阻止工具）
- **PostToolUse**: 工具成功完成后
- **UserPromptSubmit**: 用户提交提示时
- **SubagentStop**: 子代理完成时
- **Stop**: 主代理响应结束时
- **Notification**: 系统通知期间
- **PreCompact**: 上下文压缩前
- **SessionStart**: 会话初始化时

## 创建您自己的钩子

1. 编写shell脚本：
   - 从stdin读取JSON输入
   - 处理输入
   - 返回JSON输出（成功时返回空 `{}`）
   - 可返回 `{"action": "block", "message": "原因"}`来阻止操作
2. 使其可执行：

```bash
chmod +x your-hook.sh
```

3. 在settings.json中添加适当的匹配器和事件

## 安全注意事项

- 钩子执行任意shell命令
- 始终验证和清理输入
- 使用完整路径避免PATH操作
- 对文件操作需谨慎
- 部署前彻底测试钩子

## 调试钩子

使用调试标志运行Claude Code以查看钩子执行情况：

```bash
claude --debug
```

这将显示：

- 触发了哪些钩子
- 每个钩子的输入/输出
- 任何错误或问题

## 与子代理集成

示例配置包含一个与validation-gates子代理集成的钩子，展示了钩子和子代理如何协同工作，实现更健壮的开发工作流程。
