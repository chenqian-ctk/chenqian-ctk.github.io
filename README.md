# chenqian-ctk.github.io

个人博客 / 作品集，记录 AI / Agent 方向的学习笔记与小项目。

- 技术栈：[Astro](https://astro.build)
- 部署：GitHub Pages（推送到 `main` 分支自动构建发布）
- 线上地址：https://chenqian-ctk.github.io

## 本地开发

```bash
npm install
npm run dev      # 本地预览
npm run build    # 生产构建
```

## 写文章

在 `src/content/blog/` 下新建 `.md` 文件，带上如下 frontmatter 即可自动发布：

```md
---
title: "文章标题"
description: "一句话简介"
pubDate: 2026-10-10
tags: ["LangChain", "RAG"]
---
```

## 相关项目

- [langchain-rag-demo](https://github.com/chenqian-ctk/langchain-rag-demo) — LangChain 入门：本地文档问答（RAG）
- [langgraph-agent-demo](https://github.com/chenqian-ctk/langgraph-agent-demo) — LangGraph 入门：带工具的 Agent
