---
name: langchain-architecture
description: Design LLM applications using the LangChain framework with agents, memory, and tool integration patterns. Use when building LangChain applications, implementing AI agents, or creating complex LLM workflows.
---

# LangChain 架構

掌握 LangChain 框架，打造具備代理、鏈結、記憶和工具整合的精密大型語言模型應用。

## 何時使用此技能

- 建構具有工具存取能力的自主式 AI 代理
- 實作複雜的多步驟 LLM 工作流程
- 管理對話記憶與狀態
- 整合 LLM 與外部資料來源及 API
- 建立模組化、可重複使用的 LLM 應用元件
- 實作文件處理流程
- 建構生產等級的 LLM 應用

## 核心概念

### 1. 代理 (Agents)
使用 LLM 決定要採取哪些行動的自主系統。

**代理類型：**
- **ReAct**：以交錯方式進行推理與行動
- **OpenAI Functions**：利用函式呼叫 API
- **Structured Chat**：處理多輸入工具
- **Conversational**：針對聊天介面最佳化
- **Self-Ask with Search**：分解複雜查詢

### 2. 鏈結 (Chains)
對 LLM 或其他工具的一系列呼叫。

**鏈結類型：**
- **LLMChain**：基本的提示詞 + LLM 組合
- **SequentialChain**：依序執行多個鏈結
- **RouterChain**：將輸入路由到專門的鏈結
- **TransformChain**：步驟之間的資料轉換
- **MapReduceChain**：並行處理與彙總

### 3. 記憶 (Memory)
跨互動維護上下文的系統。

**記憶類型：**
- **ConversationBufferMemory**：儲存所有訊息
- **ConversationSummaryMemory**：摘要較舊的訊息
- **ConversationBufferWindowMemory**：保留最後 N 則訊息
- **EntityMemory**：追蹤關於實體的資訊
- **VectorStoreMemory**：語意相似度檢索

### 4. 文件處理
載入、轉換和儲存文件以供檢索。

**元件：**
- **Document Loaders**：從各種來源載入
- **Text Splitters**：智慧地分割文件
- **Vector Stores**：儲存和檢索嵌入向量
- **Retrievers**：取得相關文件
- **Indexes**：組織文件以高效存取

### 5. 回呼 (Callbacks)
用於記錄、監控和除錯的鉤子。

**使用案例：**
- 請求/回應記錄
- 令牌使用追蹤
- 延遲監控
- 錯誤處理
- 自訂指標收集

## 快速入門

```python
from langchain.agents import AgentType, initialize_agent, load_tools
from langchain.llms import OpenAI
from langchain.memory import ConversationBufferMemory

# Initialize LLM
llm = OpenAI(temperature=0)

# Load tools
tools = load_tools(["serpapi", "llm-math"], llm=llm)

# Add memory
memory = ConversationBufferMemory(memory_key="chat_history")

# Create agent
agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION,
    memory=memory,
    verbose=True
)

# Run agent
result = agent.run("What's the weather in SF? Then calculate 25 * 4")
```

## 架構模式

### 模式 1：使用 LangChain 的 RAG
```python
from langchain.chains import RetrievalQA
from langchain.document_loaders import TextLoader
from langchain.text_splitter import CharacterTextSplitter
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings

# Load and process documents
loader = TextLoader('documents.txt')
documents = loader.load()

text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
texts = text_splitter.split_documents(documents)

# Create vector store
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(texts, embeddings)

# Create retrieval chain
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=vectorstore.as_retriever(),
    return_source_documents=True
)

# Query
result = qa_chain({"query": "What is the main topic?"})
```

### 模式 2：帶工具的自訂代理
```python
from langchain.agents import Tool, AgentExecutor
from langchain.agents.react.base import ReActDocstoreAgent
from langchain.tools import tool

@tool
def search_database(query: str) -> str:
    """Search internal database for information."""
    # Your database search logic
    return f"Results for: {query}"

@tool
def send_email(recipient: str, content: str) -> str:
    """Send an email to specified recipient."""
    # Email sending logic
    return f"Email sent to {recipient}"

tools = [search_database, send_email]

agent = initialize_agent(
    tools,
    llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)
```

### 模式 3：多步驟鏈結
```python
from langchain.chains import LLMChain, SequentialChain
from langchain.prompts import PromptTemplate

# Step 1: Extract key information
extract_prompt = PromptTemplate(
    input_variables=["text"],
    template="Extract key entities from: {text}\n\nEntities:"
)
extract_chain = LLMChain(llm=llm, prompt=extract_prompt, output_key="entities")

# Step 2: Analyze entities
analyze_prompt = PromptTemplate(
    input_variables=["entities"],
    template="Analyze these entities: {entities}\n\nAnalysis:"
)
analyze_chain = LLMChain(llm=llm, prompt=analyze_prompt, output_key="analysis")

# Step 3: Generate summary
summary_prompt = PromptTemplate(
    input_variables=["entities", "analysis"],
    template="Summarize:\nEntities: {entities}\nAnalysis: {analysis}\n\nSummary:"
)
summary_chain = LLMChain(llm=llm, prompt=summary_prompt, output_key="summary")

# Combine into sequential chain
overall_chain = SequentialChain(
    chains=[extract_chain, analyze_chain, summary_chain],
    input_variables=["text"],
    output_variables=["entities", "analysis", "summary"],
    verbose=True
)
```

## 記憶管理最佳實務

### 選擇正確的記憶類型
```python
# For short conversations (< 10 messages)
from langchain.memory import ConversationBufferMemory
memory = ConversationBufferMemory()

# For long conversations (summarize old messages)
from langchain.memory import ConversationSummaryMemory
memory = ConversationSummaryMemory(llm=llm)

# For sliding window (last N messages)
from langchain.memory import ConversationBufferWindowMemory
memory = ConversationBufferWindowMemory(k=5)

# For entity tracking
from langchain.memory import ConversationEntityMemory
memory = ConversationEntityMemory(llm=llm)

# For semantic retrieval of relevant history
from langchain.memory import VectorStoreRetrieverMemory
memory = VectorStoreRetrieverMemory(retriever=retriever)
```

## 回呼系統

### 自訂回呼處理器
```python
from langchain.callbacks.base import BaseCallbackHandler

class CustomCallbackHandler(BaseCallbackHandler):
    def on_llm_start(self, serialized, prompts, **kwargs):
        print(f"LLM started with prompts: {prompts}")

    def on_llm_end(self, response, **kwargs):
        print(f"LLM ended with response: {response}")

    def on_llm_error(self, error, **kwargs):
        print(f"LLM error: {error}")

    def on_chain_start(self, serialized, inputs, **kwargs):
        print(f"Chain started with inputs: {inputs}")

    def on_agent_action(self, action, **kwargs):
        print(f"Agent taking action: {action}")

# Use callback
agent.run("query", callbacks=[CustomCallbackHandler()])
```

## 測試策略

```python
import pytest
from unittest.mock import Mock

def test_agent_tool_selection():
    # Mock LLM to return specific tool selection
    mock_llm = Mock()
    mock_llm.predict.return_value = "Action: search_database\nAction Input: test query"

    agent = initialize_agent(tools, mock_llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION)

    result = agent.run("test query")

    # Verify correct tool was selected
    assert "search_database" in str(mock_llm.predict.call_args)

def test_memory_persistence():
    memory = ConversationBufferMemory()

    memory.save_context({"input": "Hi"}, {"output": "Hello!"})

    assert "Hi" in memory.load_memory_variables({})['history']
    assert "Hello!" in memory.load_memory_variables({})['history']
```

## 效能最佳化

### 1. 快取
```python
from langchain.cache import InMemoryCache
import langchain

langchain.llm_cache = InMemoryCache()
```

### 2. 批次處理
```python
# Process multiple documents in parallel
from langchain.document_loaders import DirectoryLoader
from concurrent.futures import ThreadPoolExecutor

loader = DirectoryLoader('./docs')
docs = loader.load()

def process_doc(doc):
    return text_splitter.split_documents([doc])

with ThreadPoolExecutor(max_workers=4) as executor:
    split_docs = list(executor.map(process_doc, docs))
```

### 3. 串流回應
```python
from langchain.callbacks.streaming_stdout import StreamingStdOutCallbackHandler

llm = OpenAI(streaming=True, callbacks=[StreamingStdOutCallbackHandler()])
```

## 資源

- **references/agents.md**：深入探討代理架構
- **references/memory.md**：記憶系統模式
- **references/chains.md**：鏈結組合策略
- **references/document-processing.md**：文件載入與索引
- **references/callbacks.md**：監控與可觀測性
- **assets/agent-template.py**：生產就緒的代理模板
- **assets/memory-config.yaml**：記憶設定範例
- **assets/chain-example.py**：複雜鏈結範例

## 常見陷阱

1. **記憶體溢位**：未管理對話歷史長度
2. **工具選擇錯誤**：不良的工具描述會混淆代理
3. **超出上下文視窗**：超出 LLM 令牌限制
4. **缺少錯誤處理**：未捕捉和處理代理失敗
5. **低效檢索**：未最佳化向量儲存查詢

## 生產環境檢查清單

- [ ] 實作適當的錯誤處理
- [ ] 加入請求/回應記錄
- [ ] 監控令牌使用和成本
- [ ] 設定代理執行的逾時限制
- [ ] 實作速率限制
- [ ] 加入輸入驗證
- [ ] 測試邊緣案例
- [ ] 設定可觀測性（回呼）
- [ ] 實作降級策略
- [ ] 對提示詞和設定進行版本控制
