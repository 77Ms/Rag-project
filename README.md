# MemRag

基于 LangChain + Milvus 的 RAG（检索增强生成）混合检索问答系统。

## 项目结构

```
MemRag/
├── app/
│   ├── __init__.py
│   ├── api/
│   │   ├── __init__.py
│   │   └── api_service.py          # FastAPI 入口，路由/认证/会话/聊天
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config_data.py          # 配置（API Key、数据库、检索参数）
│   │   ├── knowledge_base.py       # 知识库管理（上传、语义分割、向量写入）
│   │   ├── logger.py               # 日志
│   │   ├── prompts.py              # 提示词模板
│   │   ├── rag.py                  # RAG 服务编排
│   │   └── vector_stores.py        # 混合检索（Milvus + BM25 + RRF）
│   ├── models/
│   │   ├── __init__.py
│   │   └── models.py               # 数据库模型（User, ChatSession, ChatMessage）
│   └── utils/
│       ├── __init__.py
│       ├── file_history_store.py   # 对话历史存储
│       └── file_parser.py          # 文件解析（txt/md/pdf/docx）
├── html/
│   ├── index.html                  # Vue 3 前端（登录、聊天、管理后台）
│   └── 001-man.svg                 # 默认头像
├── data/                           # 示例知识库数据
│   ├── 海绵宝宝.txt
│   ├── 精灵宝可梦.txt
│   ├── 猫和老鼠汤姆和杰瑞.txt
│   ├── 蟹黄堡制作配方.txt
│   ├── 哆啦 A 梦.txt
│   └── 火影忍者.txt
├── requirements.txt
├── LICENSE
└── README.md
```

## 核心功能

- **知识库管理**：支持 txt / md / pdf / docx 文件上传，语义分割 + 向量化，MD5 去重
- **混合检索**：Milvus 向量检索 + BM25 关键词检索，RRF 倒数秩融合
- **会话管理**：多会话隔离，AI 自动生成标题与历史摘要
- **用户系统**：注册/登录，HTTP-Only Cookie 认证
- **流式对话**：流式输出，实时检索状态反馈
- **前端界面**：Vue 3 + Tailwind CSS 单页应用

## 技术栈

| 层面 | 技术 |
|------|------|
| 后端框架 | FastAPI + Uvicorn |
| 数据库 | MySQL 8.0+（SQLAlchemy） |
| 向量存储 | Milvus Lite |
| AI 模型 | 阿里云通义千问 / DashScope Embedding |
| 检索策略 | 向量检索 + BM25 + RRF 融合 |
| 前端 | Vue 3 + Tailwind CSS |

## 环境要求

- Python 3.10+
- MySQL 8.0+

## 快速开始

### 1. 安装依赖

```bash
conda create -n rag python=3.10
conda activate rag
pip install -r requirements.txt
```

### 2. 配置

编辑 `app/core/config_data.py`，填入你的 API Key 和数据库信息：

```python
DASHSCOPE_API_KEY = '你的 DashScope API Key'
ASYNC_DATABASE_URL = 'mysql+aiomysql://用户名:密码@localhost:3306/数据库名?charset=utf8'
```

### 3. 启动

```bash
python -m app.api.api_service
```

浏览器打开 `http://localhost:8000/html/index.html`。

### 4. 上传文件（可选）

```bash
streamlit run ./app/core/app_file_uploder.py
```

## API 接口

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/auth/register` | 用户注册 |
| POST | `/auth/login` | 用户登录 |
| POST | `/sessions` | 创建会话 |
| GET | `/sessions` | 获取会话列表 |
| GET | `/chat/{session_uuid}` | 获取聊天历史 |
| POST | `/chat` | 发送消息（流式） |
| DELETE | `/delete/{session_uuid}` | 删除会话 |

## 注意事项

- 首次运行 Milvus 会自动创建 `database/milvus_db.db/`
- BM25 语料和 MD5 记录分别存储在 `database/bm25_corpus.pkl` 和 `database/md5.text`
- 日志文件位于 `logs/rag_system.log`（10MB 轮转，保留 5 个备份）
- Cookie 使用 `HttpOnly` + `SameSite=None` + `Secure`，生产环境需 HTTPS

## 许可证

MIT License