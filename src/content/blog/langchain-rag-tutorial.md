---
title: "LangChain 入门：从零写一个本地文档问答（RAG）"
description: "用最小代码理解 RAG 的五个环节：加载、切块、向量化、检索、生成。全程本地向量库 + DeepSeek，无需云端 embedding。"
pubDate: 2026-10-09
tags: ["LangChain", "RAG", "LLM"]
---

这是我 AI 学习计划的第一个项目，完整代码在 [`langchain-rag-demo`](https://github.com/chenqian-ctk/langchain-rag-demo)。

## 为什么要学 RAG

大模型有两个明显的短板：**知识有时效**（训练截止日期之后的事它不知道），以及**会一本正经地胡说八道**（幻觉）。

RAG（Retrieval-Augmented Generation，检索增强生成）的思路很朴素：回答之前，先从本地知识库里**检索**出最相关的几段文字，把它们作为上下文塞给模型，让模型"照着资料回答"。

## 五步拆解

一个最小的 RAG 就五步：

1. **加载**：读取文档（PDF / txt / md）
2. **切块**：长文切成带重叠的小片段，方便检索
3. **向量化**：每个片段用 embedding 模型转成向量
4. **检索**：把问题也向量化，找最相近的 k 个片段
5. **生成**：把检索到的上下文 + 问题一起交给 LLM 回答

## 核心代码

切块 + 入库（`ingest.py`）：

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_chroma import Chroma

splitter = RecursiveCharacterTextSplitter(chunk_size=500, chunk_overlap=50)
chunks = splitter.split_documents(docs)

Chroma.from_documents(chunks, embedding=get_embeddings(), persist_directory="./chroma_db")
```

检索 + 生成（`query.py`），用 LangChain 的 LCEL 把整条链串起来：

```python
chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | PROMPT
    | llm
    | StrOutputParser()
)
answer = chain.invoke("什么是 RAG？")
```

## 两个关键选择

- **embedding 用本地模型**（`bge-small-zh-v1.5`）而不是云端 API：离线可用、免费、数据不出本地。中文场景效果也不错。
- **LLM 用 DeepSeek**：便宜、中文友好、OpenAI 兼容，`ChatOpenAI` 换个 `base_url` 就能用。

## 踩坑与思考

- `chunk_size` / `chunk_overlap` 对检索质量影响很大：太小会切断语义，太大会引入噪声。overlap 是为了避免一句话被从中间切断。
- 检索到的片段不一定是"答案"，但一定是"相关的"。想要更准，后续可以加 rerank（重排）或混合检索。

## 下一步

RAG 是 Agent 的"查资料"能力。下一个项目我用 LangGraph 把"检索 + 工具调用 + 多步推理"串成一个真正的 Agent。
