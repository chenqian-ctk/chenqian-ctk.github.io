---
title: "LangGraph 入门：做一个会调用工具的 Agent"
description: "理解 Agent 的核心循环：LLM 决定调用哪个工具 → 执行 → 观察结果 → 再决策。用 LangGraph 把状态、节点、边建模成一张图。"
pubDate: 2026-10-08
tags: ["LangGraph", "Agent", "LLM"]
---

这是我 AI 学习计划的第二个项目，完整代码在 [`langgraph-agent-demo`](https://github.com/chenqian-ctk/langgraph-agent-demo)。

## Agent 和普通问答的区别

普通问答是"你问一句，它答一句"。Agent 则是让模型**自主决策、调用工具、多步执行**：先想清楚要干嘛，必要时调用计算器 / 搜索 / 数据库等工具，根据结果继续推理，直到产出最终答案。

## 核心：一个循环

Agent 的本质是一个循环：

```
LLM 决定调用哪个工具 → 执行工具 → 把结果交回 LLM → 再决策 → …直到给出最终答案
```

手写 `while` 循环也能实现，但状态管理和流程控制会越来越乱。LangGraph 把这个循环显式建模成一张**图**：

- **State**：Agent 的共享状态（这里是消息列表）
- **Node**：处理步骤（`chatbot` 调 LLM、`tools` 执行工具）
- **Edge**：状态流转（含条件边：有工具调用就走 `tools`，否则结束）

## 核心代码

```python
from langgraph.graph import StateGraph, START, END

class AgentState(TypedDict):
    messages: Annotated[list[BaseMessage], add_messages]

def chatbot(state):
    return {"messages": [llm_with_tools.invoke(state["messages"])]}

graph = StateGraph(AgentState)
graph.add_node("chatbot", chatbot)
graph.add_node("tools", ToolNode(ALL_TOOLS))
graph.add_edge(START, "chatbot")
graph.add_conditional_edges("chatbot", route, {"tools": "tools", END: END})
graph.add_edge("tools", "chatbot")   # 执行完回到 LLM，形成循环

agent = graph.compile()
```

## 工具是怎么"接"上去的

工具就是一个带 docstring 的普通 Python 函数，用 `@tool` 装饰器注册：

```python
@tool
def add(a: float, b: float) -> float:
    """计算两个数的和，参数 a、b 为数字。"""
    return a + b
```

LLM 靠函数名和 docstring 判断"什么时候、用什么参数"去调它——这就是 function calling。

## 踩坑与思考

- **模型必须支持 function calling**。DeepSeek 支持，但换模型前要确认。
- `add_messages` 这个 reducer 很关键：它把新消息**追加**到列表，而不是覆盖——否则 Agent 就"失忆"了。
- 工具出错是常态。生产环境要考虑重试、兜底，甚至人工审核（LangGraph 支持中断和人工介入，这是它比手写循环强的地方）。

## 下一步

把这两个项目合并：让 Agent 先走 RAG"查资料"，再调用工具、多步推理，就是一个更接近真实业务的智能体了。
