---
title: RAG檢索增強生成：向量檢索、知識庫構建
date: 2026-04-20
tags: [RAG, Retrieval-Augmented Generation, Vector Search, Knowledge Base, LLM]
description: 介紹檢索增強生成（RAG）的基本原理、向量化技術、向量檢索機制，以及企業知識庫的建構策略。
---

# RAG檢索增強生成：向量檢索、知識庫構建

## 概述

檢索增強生成（Retrieval-Augmented Generation, RAG）是一種將外部知識檢索與語言模型生成相結合的技術架構。RAG 的核心動機在於緩解 LLM 知識時效性不足與幻覺問題，同時讓模型能夠引用真實來源回答特定領域問題。Lewis et al. (2020) 在其論文中首次系統性提出 RAG 架構，成為近年 NLP 領域最具影響力的研究方向之一。

## RAG 工作流程

典型 RAG 流程可分為以下階段：

1. **知識處理（Indexing）**：將文件分塊（Chunking）、向量化後存入向量資料庫。
2. **檢索（Retrieval）**：將使用者查詢向量化，透過相似度搜尋取得最相關的 Top-K 文本塊。
3. **生成（Generation）**：將檢索結果與原始查詢組合為Prompt，交由 LLM 生成最終答案。

這三階段的設計使得 RAG 能動態存取最新資訊，而無需對模型進行重新訓練。

## 向量檢索核心技術

### 文字向量化（Embedding）

文字嵌入模型將自然語言轉換為高維向量，常見模型包括：

- **Dense Embeddings**：BERT 系列、E5、M3E 等，產生稠密向量，語義表達能力強。
- **Sparse Embeddings**：BM25、TF-IDF 等，擅長關鍵字精確匹配。

現代 RAG 系統常結合 Dense 與 Sparse 兩種向量，以兼顧語義理解與關鍵字檢索能力。

### 向量索引與相似度度量

主流向量索引演算法包括：

- **HNSW（Hierarchical Navigable Small World）**：基於圖的近似最近鄰搜尋，查詢速度快、精度高，是 FAISS 與 Milvus 的預設演算法。
- **IVF（Inverted File Index）**：倒排索引，適合大規模資料。

相似度度量通常採用 **Cosine Similarity**（餘弦相似度）或 **L2 Distance**（歐氏距離）。

### 混合檢索（Hybrid Search）

混合檢索結合向量檢索與關鍵字檢索的優勢。典型做法是使用 Reciprocal Rank Fusion（RRF）演算法對兩種檢索結果進行加權融合，顯著提升召回率。

## 知識庫建構策略

### 文件分塊（Chunking）

分塊大小直接影響檢索質量與生成效果。過大的區塊增加雜訊，過小則丢失上下文。常用策略包括：

- **固定長度分塊**：簡單直觀，但可能切断語義單元。
- **語義分塊**：依段落或標題結構自動切分，更符合人類閱讀邏輯。

### 中介元（Mid-Orchestration）

在文件處理流程中插入中介元模型，用於做文件分類、關鍵實體識別、摘要預處理等，可提升後續檢索的精確度。

### Metadata  Enrichment

為每個文本塊附加元資料（如來源、日期、標籤），可在檢索時進行屬性過濾，有效縮小候選範圍。

## 知名相關工作

- **RAG** (Lewis et al., 2020, arXiv:2005.11401)：原始 RAG 架構論文。
- **Self-RAG** (Asai et al., 2023, arXiv:2310.11511)：讓模型自主判斷何時需要檢索。
- **Dense Passage Retrieval** (Karpukhin et al., 2020, arXiv:2004.04906)：高品質向量檢索經典方法。

## 小結

RAG 透過結合向量檢索與語言模型生成，有效提升問答系統的事實準確性與可溯源性。良好的知識庫建構策略與向量檢索優化，是建構，生產級 RAG 系統的關鍵所在。

---

**參考文獻**

- Lewis, P., et al. "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks." *arXiv:2005.11401*, 2020.
- Asai, A., et al. "Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection." *arXiv:2310.11511*, 2023.
- Karpukhin, V., et al. "Dense Passage Retrieval for Open-Domain Question Answering." *arXiv:2004.04906*, 2020.
