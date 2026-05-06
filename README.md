# 🧠 RAG + GraphRAG + Agent 智能知识库问答系统

基于 LangChain + FastAPI 构建的智能知识库问答系统，融合 RAG 检索增强生成、GraphRAG 知识图谱推理和 Agent 智能体工具调用。

## ✨ 功能特性

### 📚 RAG 检索增强生成
- 向量检索（FAISS + 智谱 Embedding）
- BM25 关键词检索
- 混合检索（向量 60% + BM25 40%）
- 文档重排序（Top-N 精简）
- 查询改写（优化口语化问题）
- 自适应 RAG（多策略切换）

### 🕸️ GraphRAG 知识图谱
- 三元组自动抽取（实体-关系-实体）
- 实体归一化（消除歧义）
- Cypher 自动生成（自然语言 → Neo4j 查询）
- 智能路由（自动判断走图谱还是向量检索）

#### GraphRAG 设计理念
- **零侵入**：不改动原有 RAG 任何旧代码，只新增 `graph_rag` 模块，零风险
- **全复用**：所有文档加载、切片、LLM 调用全部复用，只写新增逻辑，工作量减半

#### GraphRAG 架构流程

```
用户提问 → /api/chat
              │
              ▼
        KGQAPipeline.ask()
              │
              ▼
        智能路由（LLM 判断）
           ╱        ╲
     实体/关系类    总结/语义类
         │              │
         ▼              ▼
   知识图谱查询     向量 RAG 检索
         │              │
         ╲            ╱
          ▼          ▼
       LLM 生成最终答案
```

### 🤖 Agent 智能体
- ReAct 模式（思考 → 行动 → 观察 循环推理）
- 任务规划器（拆分复杂问题）
- 任务执行器（逐步执行子任务）

### 🔧 MCP 多工具调度
| 工具 | 功能 |
|------|------|
| `calculator` | 数学计算 |
| `file_reader` | 文件读取 |
| `rag_retrieval` | 知识库检索 |
| `web_search` | 联网搜索 |
| `graph_rag` | 知识图谱问答 |

### 🖥️ Web 前端
- 对话界面（Markdown 渲染）
- RAG / Agent 模式切换
- 路由标签显示（图谱/向量/Agent/LLM）
- 暗色模式
- 对话导出
- 一键构建知识库
- 文件上传/管理
- 对话本地存储

## 🏗️ 项目架构

### 系统架构图

```
┌─────────────────────────────────────────────────────────────┐
│                        前端界面                              │
│  对话 │ 文件上传 │ 知识库构建 │ 暗色模式 │ 导出对话          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      FastAPI 接口层                          │
│  /api/chat │ /api/agent_chat │ /api/upload │ /api/build_kb  │
└─────────────────────────────────────────────────────────────┘
                              │
            ┌─────────────────┼─────────────────┐
            ▼                 ▼                 ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│   RAG 链路    │   │  GraphRAG    │   │ Agent 智能体  │
│               │   │              │   │               │
│ 查询改写      │   │ 三元组抽取   │   │ ReAct 循环    │
│ 混合检索      │   │ 实体归一化   │   │ 任务规划      │
│ 重排序        │   │ Cypher生成   │   │ 工具调度      │
│ LLM 生成      │   │ 智能路由     │   │               │
└───────────────┘   └───────────────┘   └───────────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      MCP 工具平台                            │
│  calculator │ file_reader │ rag_retrieval │ web_search      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                      底层组件                                │
│  FAISS 向量库 │ Neo4j 图数据库 │ LLM (MiMo/GLM)              │
└─────────────────────────────────────────────────────────────┘
```

### 目录结构

```
├── api/                    # FastAPI 接口层
│   ├── app.py              # 主应用（路由、接口）
│   ├── schema.py           # 数据模型
│   └── templates/          # 前端页面
├── agent/                  # Agent 智能体模块
│   ├── base_agent.py       # 智能体基类
│   ├── react_agent.py      # ReAct 智能体
│   ├── planner.py          # 任务规划器
│   ├── executor.py         # 任务执行器
│   └── llm/                # Agent 专用 LLM
├── chain/                  # RAG 链路模块
│   ├── rag_chain.py        # RAG 问答链
│   ├── adaptive_rag.py     # 自适应 RAG
│   └── query_rewrite.py    # 查询改写
├── graph_rag/              # 知识图谱模块
│   ├── kgqa_pipeline.py    # 图谱问答管线（含智能路由）
│   ├── triple_extract.py   # 三元组抽取
│   ├── entity_norm.py      # 实体归一化
│   ├── cypher_generator.py # Cypher 生成
│   └── neo4j_client.py     # Neo4j 客户端
├── retriever/              # 检索模块
│   ├── vector_store.py     # FAISS 向量库
│   ├── bm25_retriever.py   # BM25 检索器
│   ├── hybrid_retriever.py # 混合检索器
│   ├── rerank.py           # 重排序
│   └── embedding.py        # 嵌入模型
├── loader/                 # 文档加载器
│   ├── doc_loader.py       # DOCX 加载
│   ├── pdf_loader.py       # PDF 加载
│   ├── text_loader.py      # TXT 加载
│   └── url_loader.py       # URL 加载
├── splitter/               # 文本分块模块
│   ├── semantic_splitter.py # 语义分块
│   ├── parent_chunk.py     # 父文档分块
│   └── base_splitter.py    # 分块基类
├── mcp/                    # MCP 工具调度平台
│   ├── tool_registry.py    # 工具注册中心
│   ├── scheduler.py        # 工具调度器
│   └── tool_executor.py    # 工具执行器
├── tools/                  # 工具实现
│   ├── calc_tool.py        # 计算器
│   ├── file_tool.py        # 文件读取
│   ├── rag_tool.py         # RAG 检索
│   ├── search_tool.py      # 联网搜索
│   └── graph_rag_tool.py   # 知识图谱问答
├── llm/                    # LLM 模块
│   ├── chat_model.py       # 大模型封装
│   ├── memory.py           # 对话记忆
│   └── prompt_template.py  # 提示词模板
├── utils/                  # 工具类
│   ├── logger.py           # 日志
│   ├── text_process.py     # 文本处理
│   └── eval_metrics.py     # 评估指标
├── config.py               # 全局配置
├── main.py                 # 启动入口
└── requirements.txt        # 依赖清单
```

## 🚀 快速开始

### 1. 环境要求

- Python 3.10+
- 智谱 API Key 或小米 MiMo API Key（二选一）

### 2. 克隆项目

```bash
git clone https://github.com/your-username/your-repo.git
cd your-repo
```

### 3. 创建虚拟环境

```bash
python -m venv venv

# macOS / Linux
source venv/bin/activate

# Windows
venv\Scripts\activate
```

### 4. 安装依赖

```bash
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### 5. 配置环境变量

创建 `.env` 文件：

```env
# 小米 MiMo API（默认）
MIMO_API_KEY=your_mimo_api_key

# 或使用智谱 GLM API
ZHIPU_API_KEY=your_zhipu_api_key

# Jina AI 搜索（可选，用于联网搜索）
JINA_API_KEY=your_jina_api_key
```

### 6. 启动项目

```bash
python main.py
```

访问 `http://127.0.0.1:8000` 即可使用。

## 📡 API 接口

| 接口 | 方法 | 功能 |
|------|------|------|
| `/` | GET | 前端页面 |
| `/api/chat` | POST | RAG 问答（支持 GraphRAG 路由） |
| `/api/agent_chat` | POST | Agent 模式问答 |
| `/api/upload` | POST | 上传文档 |
| `/api/files` | GET | 文件列表 |
| `/api/files/{filename}` | DELETE | 删除文件 |
| `/api/build_kb` | POST | 一键构建知识库 |
| `/test` | GET | 测试 API 连接 |
| `/api/health` | GET | 健康检查 |

## 🔧 配置说明

在 `config.py` 中修改默认配置：

```python
# 切换 LLM 提供商
LLM_CONFIG = {
    "api_key": "your_api_key",
    "base_url": "https://api.xiaomimimo.com/v1",  # 小米 MiMo
    # "base_url": "https://open.bigmodel.cn/api/paas/v4",  # 智谱 GLM
    "model_name": "mimo-v2.5-pro",
    "temperature": 0.7,
    "max_tokens": 4096,
}
```

### Neo4j 配置（可选，GraphRAG 需要）

```python
NEO4J_URI = "bolt://localhost:7687"
NEO4J_USER = "neo4j"
NEO4J_PASSWORD = "your_password"
```

使用 Docker 启动 Neo4j：

```bash
docker run -d --name neo4j -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/your_password neo4j:latest
```

## 🛠️ 技术栈

| 类别 | 技术 |
|------|------|
| 大模型 | 小米 MiMo V2.5 / 智谱 GLM-4（OpenAI 兼容协议） |
| LLM 框架 | LangChain |
| 向量数据库 | FAISS |
| 图数据库 | Neo4j（可选） |
| 后端框架 | FastAPI + Uvicorn |
| 前端 | HTML + CSS + JavaScript（marked.js） |

## 📖 使用流程

```
1. 上传文档（PDF / DOCX / TXT / MD）
       ↓
2. 点击「构建知识库」（自动分块 + 向量化）
       ↓
3. 开始提问
       ├── RAG 模式：知识图谱/向量检索 → AI 回答
       └── Agent 模式：AI 思考 → 调用工具 → 回答
```

## 📄 License

MIT License
