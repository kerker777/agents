---
name: rag-implementation
description: 為 LLM 應用程式建構檢索增強生成（Retrieval-Augmented Generation, RAG）系統，運用向量資料庫和語意搜尋。適用於實作知識基礎的 AI、建構文件問答系統，或將 LLM 與外部知識庫整合。
---

# RAG 實作

精通檢索增強生成（Retrieval-Augmented Generation, RAG），建構能運用外部知識來源提供精準、有根據回應的 LLM 應用程式。

## 何時使用此技能

- 針對專有文件建構問答系統
- 建立具有即時、事實資訊的聊天機器人
- 實作支援自然語言查詢的語意搜尋
- 透過有根據的回應減少幻覺（hallucination）
- 使 LLM 能夠存取特定領域的知識
- 建構文件助理
- 建立具有來源引用的研究工具

## 核心元件

### 1. 向量資料庫（Vector Databases）
**目的**：高效率地儲存和檢索文件嵌入（embeddings）

**選項：**
- **Pinecone**：代管服務、可擴展、快速查詢
- **Weaviate**：開源、混合搜尋
- **Milvus**：高效能、地端部署
- **Chroma**：輕量級、易於使用
- **Qdrant**：快速、支援篩選搜尋
- **FAISS**：Meta 的函式庫、本地部署

### 2. 嵌入（Embeddings）
**目的**：將文字轉換為數值向量以進行相似度搜尋

**模型：**
- **text-embedding-ada-002**（OpenAI）：通用型、1536 維度
- **all-MiniLM-L6-v2**（Sentence Transformers）：快速、輕量級
- **e5-large-v2**：高品質、多語言
- **Instructor**：任務特定指令
- **bge-large-en-v1.5**：最先進效能（SOTA）

### 3. 檢索策略（Retrieval Strategies）
**方法：**
- **Dense Retrieval（密集檢索）**：透過嵌入進行語意相似度搜尋
- **Sparse Retrieval（稀疏檢索）**：關鍵字比對（BM25、TF-IDF）
- **Hybrid Search（混合搜尋）**：結合密集與稀疏檢索
- **Multi-Query（多重查詢）**：產生多個查詢變體
- **HyDE**：產生假設性文件

### 4. 重新排序（Reranking）
**目的**：透過重新排列結果來提升檢索品質

**方法：**
- **Cross-Encoders**：基於 BERT 的重新排序
- **Cohere Rerank**：基於 API 的重新排序
- **Maximal Marginal Relevance（MMR，最大邊際相關性）**：多樣性 + 相關性
- **LLM-based**：使用 LLM 評估相關性分數

## 快速入門

```python
from langchain.document_loaders import DirectoryLoader
from langchain.text_splitters import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma
from langchain.chains import RetrievalQA
from langchain.llms import OpenAI

# 1. Load documents
loader = DirectoryLoader('./docs', glob="**/*.txt")
documents = loader.load()

# 2. Split into chunks
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    length_function=len
)
chunks = text_splitter.split_documents(documents)

# 3. Create embeddings and vector store
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(chunks, embeddings)

# 4. Create retrieval chain
qa_chain = RetrievalQA.from_chain_type(
    llm=OpenAI(),
    chain_type="stuff",
    retriever=vectorstore.as_retriever(search_kwargs={"k": 4}),
    return_source_documents=True
)

# 5. Query
result = qa_chain({"query": "What are the main features?"})
print(result['result'])
print(result['source_documents'])
```

## 進階 RAG 模式

### 模式 1：混合搜尋
```python
from langchain.retrievers import BM25Retriever, EnsembleRetriever

# Sparse retriever (BM25)
bm25_retriever = BM25Retriever.from_documents(chunks)
bm25_retriever.k = 5

# Dense retriever (embeddings)
embedding_retriever = vectorstore.as_retriever(search_kwargs={"k": 5})

# Combine with weights
ensemble_retriever = EnsembleRetriever(
    retrievers=[bm25_retriever, embedding_retriever],
    weights=[0.3, 0.7]
)
```

### 模式 2：多重查詢檢索
```python
from langchain.retrievers.multi_query import MultiQueryRetriever

# Generate multiple query perspectives
retriever = MultiQueryRetriever.from_llm(
    retriever=vectorstore.as_retriever(),
    llm=OpenAI()
)

# Single query → multiple variations → combined results
results = retriever.get_relevant_documents("What is the main topic?")
```

### 模式 3：上下文壓縮
```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import LLMChainExtractor

compressor = LLMChainExtractor.from_llm(llm)

compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=vectorstore.as_retriever()
)

# Returns only relevant parts of documents
compressed_docs = compression_retriever.get_relevant_documents("query")
```

### 模式 4：父文件檢索器
```python
from langchain.retrievers import ParentDocumentRetriever
from langchain.storage import InMemoryStore

# Store for parent documents
store = InMemoryStore()

# Small chunks for retrieval, large chunks for context
child_splitter = RecursiveCharacterTextSplitter(chunk_size=400)
parent_splitter = RecursiveCharacterTextSplitter(chunk_size=2000)

retriever = ParentDocumentRetriever(
    vectorstore=vectorstore,
    docstore=store,
    child_splitter=child_splitter,
    parent_splitter=parent_splitter
)
```

## 文件分塊策略

### 遞迴字元文字分割器
```python
from langchain.text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    length_function=len,
    separators=["\n\n", "\n", " ", ""]  # Try these in order
)
```

### 基於 Token 的分割
```python
from langchain.text_splitters import TokenTextSplitter

splitter = TokenTextSplitter(
    chunk_size=512,
    chunk_overlap=50
)
```

### 語意分塊
```python
from langchain.text_splitters import SemanticChunker

splitter = SemanticChunker(
    embeddings=OpenAIEmbeddings(),
    breakpoint_threshold_type="percentile"
)
```

### Markdown 標題分割器
```python
from langchain.text_splitters import MarkdownHeaderTextSplitter

headers_to_split_on = [
    ("#", "Header 1"),
    ("##", "Header 2"),
    ("###", "Header 3"),
]

splitter = MarkdownHeaderTextSplitter(headers_to_split_on=headers_to_split_on)
```

## 向量儲存配置

### Pinecone
```python
import pinecone
from langchain.vectorstores import Pinecone

pinecone.init(api_key="your-api-key", environment="us-west1-gcp")

index = pinecone.Index("your-index-name")

vectorstore = Pinecone(index, embeddings.embed_query, "text")
```

### Weaviate
```python
import weaviate
from langchain.vectorstores import Weaviate

client = weaviate.Client("http://localhost:8080")

vectorstore = Weaviate(client, "Document", "content", embeddings)
```

### Chroma（本地）
```python
from langchain.vectorstores import Chroma

vectorstore = Chroma(
    collection_name="my_collection",
    embedding_function=embeddings,
    persist_directory="./chroma_db"
)
```

## 檢索最佳化

### 1. 中繼資料篩選
```python
# Add metadata during indexing
chunks_with_metadata = []
for i, chunk in enumerate(chunks):
    chunk.metadata = {
        "source": chunk.metadata.get("source"),
        "page": i,
        "category": determine_category(chunk.page_content)
    }
    chunks_with_metadata.append(chunk)

# Filter during retrieval
results = vectorstore.similarity_search(
    "query",
    filter={"category": "technical"},
    k=5
)
```

### 2. 最大邊際相關性
```python
# Balance relevance with diversity
results = vectorstore.max_marginal_relevance_search(
    "query",
    k=5,
    fetch_k=20,  # Fetch 20, return top 5 diverse
    lambda_mult=0.5  # 0=max diversity, 1=max relevance
)
```

### 3. 使用 Cross-Encoder 重新排序
```python
from sentence_transformers import CrossEncoder

reranker = CrossEncoder('cross-encoder/ms-marco-MiniLM-L-6-v2')

# Get initial results
candidates = vectorstore.similarity_search("query", k=20)

# Rerank
pairs = [[query, doc.page_content] for doc in candidates]
scores = reranker.predict(pairs)

# Sort by score and take top k
reranked = sorted(zip(candidates, scores), key=lambda x: x[1], reverse=True)[:5]
```

## RAG 提示工程

### 上下文提示
```python
prompt_template = """Use the following context to answer the question. If you cannot answer based on the context, say "I don't have enough information."

Context:
{context}

Question: {question}

Answer:"""
```

### 帶引用的提示
```python
prompt_template = """Answer the question based on the context below. Include citations using [1], [2], etc.

Context:
{context}

Question: {question}

Answer (with citations):"""
```

### 帶信心度的提示
```python
prompt_template = """Answer the question using the context. Provide a confidence score (0-100%) for your answer.

Context:
{context}

Question: {question}

Answer:
Confidence:"""
```

## 評估指標

```python
def evaluate_rag_system(qa_chain, test_cases):
    metrics = {
        'accuracy': [],
        'retrieval_quality': [],
        'groundedness': []
    }

    for test in test_cases:
        result = qa_chain({"query": test['question']})

        # Check if answer matches expected
        accuracy = calculate_accuracy(result['result'], test['expected'])
        metrics['accuracy'].append(accuracy)

        # Check if relevant docs were retrieved
        retrieval_quality = evaluate_retrieved_docs(
            result['source_documents'],
            test['relevant_docs']
        )
        metrics['retrieval_quality'].append(retrieval_quality)

        # Check if answer is grounded in context
        groundedness = check_groundedness(
            result['result'],
            result['source_documents']
        )
        metrics['groundedness'].append(groundedness)

    return {k: sum(v)/len(v) for k, v in metrics.items()}
```

## 資源

- **references/vector-databases.md**：向量資料庫詳細比較
- **references/embeddings.md**：嵌入模型選擇指南
- **references/retrieval-strategies.md**：進階檢索技術
- **references/reranking.md**：重新排序方法及使用時機
- **references/context-window.md**：管理上下文限制
- **assets/vector-store-config.yaml**：配置範本
- **assets/retriever-pipeline.py**：完整 RAG 流程
- **assets/embedding-models.md**：模型比較和基準測試

## 最佳實踐

1. **分塊大小**：在上下文與特定性之間取得平衡（500-1000 tokens）
2. **重疊**：使用 10-20% 重疊以保留邊界的上下文
3. **中繼資料**：包含來源、頁碼、時間戳記以便篩選和除錯
4. **混合搜尋**：結合語意搜尋和關鍵字搜尋以獲得最佳結果
5. **重新排序**：使用 cross-encoder 改善最佳結果
6. **引用**：始終回傳來源文件以確保透明度
7. **評估**：持續測試檢索品質和回答準確性
8. **監控**：在生產環境中追蹤檢索指標

## 常見問題

- **檢索不佳**：檢查嵌入品質、分塊大小、查詢表達方式
- **不相關的結果**：新增中繼資料篩選、使用混合搜尋、重新排序
- **資訊遺漏**：確保文件已正確索引
- **查詢緩慢**：最佳化向量儲存、使用快取、減少 k 值
- **幻覺**：改善基礎提示、新增驗證步驟
