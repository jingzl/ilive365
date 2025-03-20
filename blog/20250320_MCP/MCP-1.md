## 1. 概述
**MCP**（Model Context Protocol，模型上下文协议） 是由 Anthropic 推出的一种开放标准，旨在统一大型语言模型（LLM）与外部数据源和工具之间的通信协议。MCP 的主要目的在于解决当前 AI 模型因数据孤岛限制而无法充分发挥潜力的难题，MCP 使得 AI 应用能够安全地访问和操作本地及远程数据，为 AI 应用提供了连接万物的接口。
**官网**：[Model Context Protocol](https://github.com/modelcontextprotocol)，[Introduction - Model Context Protocol](https://modelcontextprotocol.io/introduction)
**SPEC**：[Specification (Latest) – Model Context Protocol Specification](https://spec.modelcontextprotocol.io/specification/2024-11-05/)
**SDK支持**：目前支持的版本有：
- Python：[modelcontextprotocol/python-sdk: The official Python SDK for Model Context Protocol servers and clients](https://github.com/modelcontextprotocol/python-sdk)
- TypeScript：[modelcontextprotocol/typescript-sdk: The official Typescript SDK for Model Context Protocol servers and clients](https://github.com/modelcontextprotocol/typescript-sdk)
- Java：[modelcontextprotocol/java-sdk: The official Java SDK for Model Context Protocol servers and clients. Maintained in collaboration with Spring AI](https://github.com/modelcontextprotocol/java-sdk)
- Kotlin：[modelcontextprotocol/kotlin-sdk: The official Kotlin SDK for Model Context Protocol servers and clients. Maintained in collaboration with JetBrains](https://github.com/modelcontextprotocol/kotlin-sdk)

## 2. 整体架构
### 2.1 MCP协议架构
MCP 遵循客户端-服务器架构（client-server），支持多种数据形式的交换，并具有良好的可扩展性。内置了安全机制以确保资源分配的安全控制，并且目前支持本地运行。官方计划未来增加对企业级身份验证的支持，以允许远程服务器的操作。Anthropic公司希望通过推动MCP成为行业开放标准，类似于HTTP对于网络浏览器和服务器间信息交换的作用。这样的标准化努力已经得到了一些合作伙伴的认可和支持，包括金融支付公司Block和数据管理解决方案供应商Apollo。此外，Anthropic还获得了亚马逊40亿美元的投资，用于加强其企业市场的服务，特别是针对企业客户的AI模型训练和部署。市场上也有声音质疑MCP能否真正成为广泛接受的标准，担心这可能会导致AI生态系统的进一步碎片化。然而，Anthropics的努力显示了他们致力于将MCP构建为开源生态系统，邀请更多的AI工具开发者和企业加入，共同促进AI技术的发展和应用。

[图]

**基础协议**
All messages between MCP clients and servers MUST follow the JSON-RPC 2.0 specification. The protocol defines three fundamental types of messages:
[图]

**通信机制**
MCP（Model Context Protocol）的连接机制遵循客户端-服务器架构。在这种架构中，MCP Clients与MCP Servers之间建立一对一的连接。
这种设计允许MCP Hosts（如AI应用程序）通过MCP Clients与一个或多个MCP Servers进行通信，以获取数据和执行任务。
MCP支持两种类型的通信机制：
- 标准输入输出（Stdio）：适用于本地进程间通信，其中Client启动Server程序作为子进程，消息通讯通过stdin/stdout进行，消息格式为JSON-RPC 2.0。
- 服务器发送事件（SSE）：用于基于HTTP的通信，允许服务器向客户端推送消息，而客户端到服务器的消息传递则使用HTTP POST，同样采用JSON-RPC 2.0格式进行消息交换。
所有传输都使用JSON-RPC 2.0进行消息交换，这为MCP Clients和MCP Servers之间的通信提供了统一的消息格式。至于连接类型，MCP没有明确指出是长连接还是短连接，但考虑到其基于JSON-RPC 2.0的特性，它更可能支持长连接，以便保持客户端和服务器之间的持久交互状态。MCP的通信协议可以是TCP或UDP，这取决于具体的实现和部署需求。

### 2.2 核心概念
- MCP 主机（MCP Hosts）：发起请求的 LLM 应用程序（例如 Claude Desktop、IDE 或 AI 工具）。
- MCP 客户端（MCP Clients）：在主机程序内部，与 MCP server 保持 1:1 的连接。
- MCP 服务器（MCP Servers）：为 MCP client 提供上下文、工具和 prompt 信息。
- 本地资源（Local Resources）：本地计算机中可供 MCP server 安全访问的资源（例如文件、数据库）。
- 远程资源（Remote Resources）：MCP server 可以连接到的远程资源（例如通过 API）。

### 2.3 MCP Server
MCP Server 是 MCP 架构中的关键组件，Servers provide the fundamental building blocks for adding context to language models via MCP. These primitives enable rich interactions between clients, servers, and language models:
- Prompts: Pre-defined templates or instructions that guide language model interactions
- Resources: Structured data or content that provides additional context to the model
- Tools: Executable functions that allow models to perform actions or retrieve information

[图]

资源（Resources），工具（Tools）和 提示（Prompts），这些功能使 MCP server 能够为 AI 应用提供丰富的上下文信息和操作能力，从而增强 LLM 的实用性和灵活性。
可以在 [MCP Servers Repository](https://github.com/modelcontextprotocol/servers) 和 [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers) 中找到许多由社区实现的 MCP server。使用 TypeScript 编写的 MCP server 可以通过 npx 命令来运行，使用 Python 编写的 MCP server 可以通过 uvx 命令来运行。

### 2.4 MCP Client
MCP client 充当 LLM 和 MCP server 之间的桥梁，MCP client 的工作流程如下：
- MCP client 首先从 MCP server 获取可用的工具列表。
- 将用户的查询连同工具描述通过 function calling 一起发送给 LLM。
- LLM 决定是否需要使用工具以及使用哪些工具。
- 如果需要使用工具，MCP client 会通过 MCP server 执行相应的工具调用。
- 工具调用的结果会被发送回 LLM。
- LLM 基于所有信息生成自然语言响应。
- 最后将响应展示给用户。
在 [Example Clients - Model Context Protocol](https://modelcontextprotocol.io/clients) 找到当前支持 MCP 协议的客户端程序。
以下是部分支持的应用：
[图]

---
