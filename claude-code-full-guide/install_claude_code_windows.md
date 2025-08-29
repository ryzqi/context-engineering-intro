## 在 Windows 上安装 Claude Code（使用 WSL）

Claude Code 默认仅支持 Linux 和 MacOS。要在 Windows 上使用 Claude Code，您可以使用 WSL。

1. 前往 Microsoft Store
2. 搜索 Ubuntu WSL 并安装
3. 在终端中打开 WSL
4. 运行以下命令（遵循最佳安全实践）：

```bash
# 首先，保存现有全局包列表以便后续迁移
npm list -g --depth=0 > ~/npm-global-packages.txt

# 创建全局包目录
mkdir -p ~/.npm-global

# 配置 npm 使用新的目录路径
npm config set prefix ~/.npm-global

# 注意：将 ~/.bashrc 替换为 ~/.zshrc、~/.profile 或其他适用于您 shell 的文件
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc

# 应用新的 PATH 设置
source ~/.bashrc

# 现在在新位置重新安装 Claude Code
npm install -g @anthropic-ai/claude-code
```

5. 现在，在您的 IDE 中可以通过 Ctrl + J 打开终端（也可使用此快捷键关闭），点击加号旁边的下拉箭头以打开 Ubuntu (WSL) 终端，然后运行 "claude" 命令启动 Claude Code。
