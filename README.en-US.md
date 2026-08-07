

# 🧠 RAG + GraphRAG + Agent Intelligent Knowledge Base Q&A System

An intelligent knowledge base Q&A system built on LangChain + FastAPI, integrating RAG (Retrieval-Augmented Generation), GraphRAG (Knowledge Graph Reasoning), and Agent (AI Agent Tool Invocation).

## ✨ Features

### 📚 RAG (Retrieval-Augmented Generation)
- Vector Search (FAISS + Zhipu Embedding)
- BM25 Keyword Search
- Hybrid Search (60% Vector + 40% BM25)
- Document Reranking (Top-N Refinement)
- Query Rewriting (Optimizing Conversational Queries)
- Adaptive RAG (Multi-Strategy Switching)

### 🕸️ GraphRAG Knowledge Graph
- Automatic Triple Extraction (Entity-Relation-Entity)
- Entity Normalization (Resolving Ambiguity)
- Automatic Cypher Generation (Natural Language → Neo4j Query)
- Intelligent Routing (Automatically determining whether to use Graph or Vector Search)

#### GraphRAG Design Philosophy
- **Zero Intrusion**: Does not modify any existing RAG code. Only adds the `graph_rag` module, ensuring zero risk.
- **Full Reusability**: Reuses all document loading, chunking, and LLM calls. Only new logic needs to be written, halving the development workload.

#### GraphRAG Architecture Workflow

```
User Query → /api/chat
             │
             ▼
      KGQAPipeline.ask()
             │
             ▼
      Intelligent Routing (LLM Judgment)
         ╱        ╲
Entity/Relation   Summary/Semantic
     │              │
     ▼              ▼
Knowledge Graph   Vector RAG Search
     │              │
     ╲            ╱
      ▼          ▼
    LLM Generates Final Answer
```

### 🤖 Agent (AI Agent)
- ReAct Mode (Thinking → Acting → Observing Loop)
- Task Planner (Decomposing Complex Problems)
- Task Executor (Executing Subtasks Step-by-Step)

### 🔧 MCP Multi-Tool Orchestration
| Tool | Function |
|------|----------|
| `calculator` | Math Calculation |
| `file_reader` | File Reading |
| `rag_retrieval` | Knowledge Base Retrieval |
| `web_search` | Web Search |
| `graph_rag` | Knowledge Graph Q&A |

### 🖥️ Web Frontend
- Chat Interface (Markdown Rendering)
- RAG / Agent Mode Switching
- Routing Tag Display (Graph/Vector/Agent/LLM)
- Dark Mode
- Chat Export
- One-Click Knowledge Base Building
- File Upload/Management
- Local Chat Storage

## 🏗️ Project Architecture

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                      Frontend UI                            │
│  Chat │ File Upload │ KB Build │ Dark Mode │ Export Chat   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   FastAPI API Layer                         │
│  /api/chat │ /api/agent_chat │ /api/upload │ /api/build_kb  │
└─────────────────────────────────────────────────────────────┘
                              │
            ┌─────────────────┼─────────────────┐
            ▼                 ▼                 ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│    RAG Chain  │   │  GraphRAG    │   │  AI Agent     │
│               │   │              │   │               │
│  Query Rewrite│   │ Triple Extract│   │  ReAct Loop   │
│  Hybrid Search│   │ Entity Norm   │   │  Task Planning│
│  Reranking    │   │ Cypher Gen    │   │  Tool SchedOrchest│
│  LLM Gen      │   │ Smart Route   │   │               │
└───────────────┘   └───────────────┘   └───────────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                  MCP Tool Platform                          │
│  calculator │ file_reader │ rag_retrieval │ web_search      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                  Underlying Components                      │
│  FAISS Vector DB │ Neo4j Graph DB │ LLM (MiMo/GLM)         │
└─────────────────────────────────────────────────────────────┘
```

### Directory Structure

```
├── api/                    # FastAPI API Layer
│   ├── app.py              # Main application (routing, endpoints)
│   ├── schema.py           # Data models
│   └── templates/          # Frontend pages
├── agent/                  # AI Agent Module
│   ├── base_agent.py       # Agent base class
│   ├── react_agent.py      # ReAct Agent
│   ├── planner.py          # Task planner
│   ├── executor.py         # Task executor
│   └── llm/                # Agent-specific LLM
├── chain/                  # RAG Chain Module
│   ├── rag_chain.py        # RAG Q&A chain
│   ├── adaptive_rag.py     # Adaptive RAG
│   └── query_rewrite.py    # Query rewriting
├── graph_rag/              # Knowledge Graph Module
│   ├── kgqa_pipeline.py    # Graph Q&A pipeline (with intelligent routing)
│   ├── triple_extract.py   # Triple extraction
│   ├── entity_norm.py      # Entity normalization
│   ├── cypher_generator.py # Cypher generation
│   └── neo4j_client.py     # Neo4j client
├── retriever/              # Retriever Module
│   ├── vector_store.py     # FAISS vector store
│   ├── bm25_retriever.py   # BM25 retriever
│   ├── hybrid_retriever.py # Hybrid retriever
│   ├── rerank.py           # Reranking
│   └── embedding.py        # Embedding model
├── loader/                 # Document Loader Module
│   ├── doc_loader.py       # DOCX loader
│   ├── pdf_loader.py       # PDF loader
│   ├── text_loader.py      # TXT loader
│   └── url_loader.py       # URL loader
├── splitter/               # Text Chunking Module
│   ├── semantic_splitter.py # Semantic chunking
│   ├── parent_chunk.py     # Parent chunking
│   └── base_splitter.py    # Chunking base class
├── mcp/                    # MCP Tool Orchestration Platform
│   ├── tool_registry.py    # Tool registry center
│   ├── scheduler.py        # Tool scheduler
│   └── tool_executor.py    # Tool executor
├── tools/                  # Tool Implementations
│   ├── calc_tool.py        # Calculator
│   ├── file_tool.py        # File reader
│   ├── rag_tool.py         # RAG retrieval
│   ├── search_tool.py      # Web search
│   └── graph_rag_tool.py   # Knowledge graph Q&A
├── llm/                    # LLM Module
│   ├── chat_model.py       # LLM wrapper
│   ├── memory.py           # Conversation memory
│   └── prompt_template.py  # Prompt templates
├── utils/                  # Utility Classes
│   ├── logger.py           # Logging
│   ├── text_process.py     # Text processing
│   └── eval_metrics.py     # Evaluation metrics
├── config.py               # Global configuration
├── main.py                 # Startup entry
└── requirements.txt        # Dependency list
```

## 🚀 Quick Start

### 1. Environment Requirements

- Python 3.10+
- Zhipu API Key or Xiaomi MiMo API Key (choose one)

### 2. Clone the Project

```bash
git clone https://github.com/your-username/your-repo.git
cd your-repo
```

### 3. Create a Virtual Environment

```bash
python -m venv venv

# macOS / Linux
source venv/bin/activate

# Windows
venv\Scripts\activate
```

### 4. Install Dependencies

```bash
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### 5. Configure Environment Variables

Create a `.env` file:

```env
# Xiaomi MiMo API (default)
MIMO_API_KEY=your_mimo_api_key

# Or use Zhipu GLM API
ZHIPU_API_KEY=your_zhipu_api_key

# Jina AI Search (optional, for web search)
JINA_API_KEY=your_jina_api_key
```

### 6. Run the Project

```bash
python main.py
```

Visit `http://127.0.0.1:8000` to get started.

## 📡 API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/` | GET | Frontend Page |
| `/api/chat` | POST | RAG Q&A (Supports GraphRAG Routing) |
| `/api/agent_chat` | POST | Agent Mode Q&A |
| `/api/upload` | POST | Upload Document |
| `/api/files` | GET | File List |
| `/api/files/{filename}` | DELETE | Delete File |
| `/api/build_kb` | POST | One-Click KB Build |
| `/test` | GET | Test API Connection |
| `/api/health` | GET | Health Check |

## 🔧 Configuration Instructions

Modify default configurations in `config.py`:

```python
# Switch LLM Provider
LLM_CONFIG = {
    "api_key": "your_api_key",
    "base_url": "https://api.xiaomimimo.com/v1",  # Xiaomi MiMo
    # "base_url": "https://open.bigmodel.cn/api/paas/v4",  # Zhipu GLM
    "model_name": "mimo-v2.5-pro",
    "temperature": 0.7,
    "max_tokens": 4096,
}
```

### Neo4j Configuration (Optional, required for GraphRAG)

```python
NEO4J_URI = "bolt://localhost:7687"
NEO4J_USER = "neo4j"
NEO4J_PASSWORD = "your_password"
```

Start Neo4j using Docker:

```bash
docker run -d --name neo4j -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/your_password neo4j:latest
```

## 🛠️ Tech Stack

| Category | Technology |
|----------|------------|
| Large Model | Xiaomi MiMo V2.5 / Zhipu GLM-4 (OpenAI Compatible Protocol) |
| LLM Framework | LangChain |
| Vector Database | FAISS |
| Graph Database | Neo4j (Optional) |
| Backend Framework | FastAPI + Uvicorn |
| Frontend | HTML + CSS + JavaScript (marked.js) |

## 📖 Usage Workflow

```
1. Upload Documents (PDF / DOCX / TXT / MD)
       ↓
2. Click "Build Knowledge Base" (Automatic Chunking + Vectorization)
       ↓
3. Start Asking Questions
       ├── RAG Mode: Knowledge Graph/Vector Search → AI Answer
       └── Agent Mode: AI Thinking → Call Tools → Answer
```

## 📄 License

MIT License
