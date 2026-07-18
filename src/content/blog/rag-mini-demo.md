---
title: '用 60 行 Python 搭一个最小 RAG 检索原型'
description: '不依赖任何框架，从零实现一个"向量化 + 检索 + 拼 Prompt"的最小 RAG 原型，理清检索增强生成的核心链路。'
pubDate: 2026-07-17
tags: ['RAG', 'LLM', 'Python']
---

检索增强生成（RAG）听着唬人，但核心链路就三步：**把文档切块向量化 → 用户提问时检索最相关的块 → 拼进 Prompt 交给大模型**。这篇用 60 行纯 Python 走一遍，不引任何框架，目的是把"黑盒"拆开看清楚。

## 1. 准备依赖

只需要两个库：一个做向量化，一个调大模型（这里用 OpenAI 兼容接口，换成任意供应商都行）。

```bash
pip install openai numpy
```

## 2. 切块与向量化

真实场景里文档可能很长，先按固定大小切块，再逐块向量化。这里用最简单的滑动窗口切分：

```python
import numpy as np
from openai import OpenAI

client = OpenAI()  # 读取环境变量 OPENAI_API_KEY

def embed(text: str) -> list[float]:
    resp = client.embeddings.create(
        model="text-embedding-3-small",
        input=text,
    )
    return resp.data[0].embedding

def chunk(text: str, size: int = 200, overlap: int = 40) -> list[str]:
    """按字符滑动窗口切块，overlap 保留跨块语义连续性。"""
    chunks = []
    start = 0
    while start < len(text):
        chunks.append(text[start : start + size])
        start += size - overlap
    return [c for c in chunks if c.strip()]

docs = chunk(open("kb.txt", encoding="utf-8").read())
index = np.array([embed(c) for c in docs])  # shape: (n_chunks, dim)
```

## 3. 检索最相关的块

用余弦相似度排个序，取 Top-K。注意先对向量做 L2 归一化，点积即余弦相似度：

```python
def retrieve(query: str, k: int = 3) -> list[str]:
    q = np.array(embed(query))
    q = q / np.linalg.norm(q)
    sims = index @ q  # 已归一化，点积=余弦
    top = np.argsort(-sims)[:k]
    return [docs[i] for i in top]
```

## 4. 拼 Prompt 交给大模型

把检索到的上下文塞进系统提示，剩下的就是一次普通对话补全：

```python
def ask(query: str) -> str:
    context = "\n\n".join(retrieve(query))
    resp = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "你是严谨的助手，只基于下面的上下文回答。"},
            {"role": "user", "content": f"上下文：\n{context}\n\n问题：{query}"},
        ],
    )
    return resp.choices[0].message.content

print(ask("我们的退款政策是怎样的？"))
```

## 5. 工程化时你会踩的坑

最小原型跑通后，真正落地要注意这几件事：

- **块大小**：太大检索不精准，太小丢失上下文，通常 200–500 字 + 少量 overlap 起步，再按评测调。
- **元数据过滤**：给每个块带上来源/时间/权限标签，检索时先按标签筛，再算相似度。
- **重排（rerank）**：向量召回 Top-20，再用一个 cross-encoder 重排取 Top-3，质量明显提升。
- **评估**：别只看 demo 效果，用一批真实问答做召回率 / 答案准确率回归测试。

> 这个原型没有用任何向量数据库，全量余弦检索是 O(n)。当文档到十万级时，再换成 FAISS / pgvector 之类的 ANN 索引——但链路逻辑完全一样。

最小 RAG 的价值，不在于框架多高级，而在于你清楚每一行在干什么。框架只是把上面这四步封装得更稳、更快而已。