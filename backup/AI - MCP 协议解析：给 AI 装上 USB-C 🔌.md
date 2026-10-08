> 2025 年开年最热的开发者话题不是某个大模型，而是一个协议：MCP（Model Context Protocol）。Anthropic 2024 年底开源它时没人想到，半年后 Cursor、Claude Code、Cline 全在接它。这篇把它的设计思想拆开讲透。

## 它到底解决了什么

大模型的上下文是封闭的：它不知道你的文件、数据库、API。之前每个 AI 应用都要自己写一套 tool calling 胶水代码，N 个模型 × M 个数据源 = N×M 的适配地狱。

MCP 把这个变成了 N+M：数据源按协议暴露能力，模型按协议消费。用 Anthropic 自己的比喻：**MCP 是 AI 应用的 USB-C**。

## 三层架构

```
Host（Claude Code / Cursor）
 └─ Client（一对一连接）
     └─ Server（本地进程 / 远程服务）
```

- **Host**：用户交互的 AI 应用，管理多个 Client。
- **Client**：协议客户端，1:1 对应一个 Server，负责能力协商。
- **Server**：能力提供方，可以是一个本地 stdio 进程，也可以是远程 SSE 服务。

传输层最初只有两种：**stdio**（本地子进程，简单可信）和 **SSE**（远程服务）。JSON-RPC 2.0 做消息信封——选型很务实：调试时直接看 JSON 明文。

## 三种原语，权限边界是精髓

这是 MCP 设计最漂亮的地方，三种原语对应三种控制权：

- **Tools**：由模型自主决定调用。`list_tools` → 模型看到 schema → 自主 `call_tool`。这是 Agent 的手脚。
- **Resources**：由应用（Host）控制暴露。文件、数据库记录，模型只能读应用给它的。是 Agent 的眼睛。
- **Prompts**：由用户触发的预置模板。是 Agent 的起手式。

外加 **Roots**（工作区边界）、**Sampling**（Server 反向请求模型补全，用于 agentic 工作流）。

注意这个权限划分：**谁控制、谁决策、谁触发，三权分立**。大部分后来的"AI 协议"都没想清楚这一层。

## 为什么是它赢了

1. **时机**：正好卡在 Agent 编程爆发的窗口期（2025 春），Cursor 们急需标准。
2. **足够简单**：JSON-RPC + 三原语，一个周末就能写出 Server。
3. **Anthropic 背书**：Claude 是当时 tool calling 最强的模型，跟着强者走。

## 阴影：安全问题

2025 年 4 月社区已经在吵了：

- **Tool squatting**：恶意 Server 注册和知名工具同名的 tool，模型分不清。
- **Prompt injection 经由 Resources**：Server 返回的"数据"里藏指令，模型照做。
- **权限过大**：很多 Server 一上来就要文件系统全权。

MCP 解决了互联互通，但**信任模型**还没解决。协议只定义了"怎么说话"，没定义"该不该信"。这会是它下一阶段最大的坑。

## 一句话总结

MCP 的胜利是"简单协议 + 好时机"的胜利。它没有发明新技术，只是把 N×M 的胶水问题变成了标准接口——而历史上所有伟大的协议（HTTP、USB）都是这么赢的。接下来要看的，是安全和权限模型能不能跟上它的扩张速度。

##{"timestamp":1744084800,"style":".markdown-body>:last-child{display:none}"}