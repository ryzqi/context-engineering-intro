## 功能特性：

- 一个 Pydantic AI agent，它拥有另一个 Pydantic AI agent 作为工具。
- 主 agent 是一个 Research Agent，子 agent 是一个 email draft Agent。
- 用于与 agent 交互的 CLI。
- email draft agent 使用 Gmail，research agent 使用 Brave API。

## 示例：

在 `examples/` 文件夹中有一个 README，你可以阅读它来理解示例的全部内容，以及在为上述功能特性创建文档时如何构建你自己的 README。

- `examples/cli.py` - 使用此文件作为模板来创建 CLI
- `examples/agent/` - 通读此文件夹中的所有文件，以了解创建支持不同 provider 和 LLM 的 Pydantic AI agent、处理 agent 依赖以及向 agent 添加工具的最佳实践。

不要直接复制任何这些示例，它们完全是为另一个项目准备的。但可以将其用作灵感和最佳实践的参考。

## 文档：

Pydantic AI 文档：https://ai.pydantic.dev/

## 其他注意事项：

- 包含一个 `.env.example` 和一个 README，其中包含设置说明，包括如何配置 Gmail 和 Brave。
- 在 README 中包含项目结构。
- 虚拟环境已设置好，并包含了必要的依赖项。
- 使用 `python_dotenv` 和 `load_env()` 来处理环境变量。
