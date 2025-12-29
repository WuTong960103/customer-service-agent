# 晨星家居智能客服助手

基于LangChain和Qwen大模型的智能客服系统，支持意图识别、槽位填充、RAG知识检索和多轮对话。

## 项目结构

```
ai智能客服/
├── backend/                    # 后端服务
│   ├── app/
│   │   ├── api/               # API接口
│   │   │   ├── chat.py        # 聊天相关接口
│   │   │   └── data.py        # 数据导入接口
│   │   ├── core/              # 核心配置
│   │   │   ├── config.py      # 应用配置
│   │   │   └── logger.py      # 日志配置
│   │   ├── models/            # 数据模型
│   │   │   └── schemas.py     # Pydantic模型
│   │   ├── services/          # 业务服务
│   │   │   ├── agent.py       # Agent核心逻辑
│   │   │   ├── conversation.py # 对话管理
│   │   │   ├── embedding.py   # 向量化服务
│   │   │   ├── intent.py      # 意图识别
│   │   │   └── rag.py         # RAG知识检索
│   │   ├── tools/             # Agent工具
│   │   │   └── business_tools.py
│   │   ├── utils/             # 工具函数
│   │   │   └── generate_sample_data.py
│   │   └── main.py            # FastAPI入口
│   ├── data/                  # 数据文件
│   ├── logs/                  # 日志文件
│   ├── requirements.txt       # Python依赖
│   └── start.sh              # 启动脚本
├── frontend/                  # 前端服务
│   ├── src/
│   │   ├── assets/           # 静态资源
│   │   ├── stores/           # Pinia状态管理
│   │   ├── App.vue           # 主组件
│   │   └── main.js           # 入口文件
│   ├── package.json          # Node依赖
│   └── start.sh              # 启动脚本
└── README.md
```

## 技术栈

### 后端
- Python 3.10+
- FastAPI (Web框架)
- LangChain (LLM框架)
- Qwen (大语言模型)
- Redis (对话历史管理)
- ChromaDB (向量数据库)
- Ollama BGE-M3 (向量化模型)

### 前端
- Vue 3
- Vite
- Pinia
- SCSS

## 功能特性

1. **意图识别与槽位填充**
   - 基于向量相似度检索候选意图
   - LLM精确匹配并提取槽位信息
   - 支持多轮对话中的意图延续

2. **主动澄清机制**
   - 检测模糊表达和指示代词
   - 智能生成澄清问题
   - 限制最大澄清次数后转人工

3. **RAG知识检索**
   - 基于ChromaDB的向量检索
   - 支持知识库动态导入
   - 检索增强的答案生成

4. **工具调用（Function Call）**
   - 订单查询
   - 物流追踪
   - 退货退款申请
   - 地址修改
   - 发票申请
   - 转人工服务

5. **对话管理**
   - Redis存储对话历史
   - 用户隔离
   - 会话管理（创建/切换/删除）
   - 流式输出响应

## 环境要求

- Python 3.10+
- Node.js 18+
- Redis
- Ollama (运行BGE-M3模型)

## 快速开始

### 1. 准备环境

确保已安装并启动：
- Redis服务（默认localhost:6379，无密码）
- Ollama并拉取bge-m3模型：`ollama pull bge-m3`

设置环境变量：
```bash
export DASHSCOPE_API_KEY=your_api_key
```

### 2. 启动后端

```bash
cd backend
./start.sh
```

首次启动会自动：
- 创建Python虚拟环境
- 安装依赖
- 生成示例数据

后端服务地址：http://localhost:8000
API文档：http://localhost:8000/docs

### 3. 启动前端

```bash
cd frontend
./start.sh
```

首次启动会自动安装npm依赖。

前端服务地址：http://localhost:3000

### 4. 导入测试数据

项目提供了测试数据文件，位于 `backend/app/data/` 目录：
- `intent_templates.xlsx` - 意图模板数据
- `knowledge_base.xlsx` - 知识库数据

启动后端后，有以下几种方式导入数据：

#### 方式一：使用 curl 命令行

```bash
# 在项目根目录执行

# 导入意图模板
curl -X POST "http://localhost:8000/api/v1/data/intents/import" \
  -F "file=@backend/app/data/intent_templates.xlsx"

# 导入知识库
curl -X POST "http://localhost:8000/api/v1/data/knowledge/import" \
  -F "file=@backend/app/data/knowledge_base.xlsx"
```

#### 方式二：使用 Swagger UI

1. 打开浏览器访问 http://localhost:8000/docs
2. 找到 **数据管理** 分组
3. 展开 `POST /api/v1/data/intents/import` 接口
4. 点击 "Try it out" 按钮
5. 点击 "Choose File" 选择 `backend/app/data/intent_templates.xlsx`
6. 点击 "Execute" 执行导入
7. 同样的方式导入知识库文件

#### 方式三：使用 Python 脚本

```python
import requests

base_url = "http://localhost:8000/api/v1/data"

# 导入意图模板
with open("backend/app/data/intent_templates.xlsx", "rb") as f:
    resp = requests.post(f"{base_url}/intents/import", files={"file": f})
    print(resp.json())

# 导入知识库
with open("backend/app/data/knowledge_base.xlsx", "rb") as f:
    resp = requests.post(f"{base_url}/knowledge/import", files={"file": f})
    print(resp.json())
```

#### 验证导入结果

```bash
# 查看意图数据数量
curl "http://localhost:8000/api/v1/data/intents/count"

# 查看知识库数量
curl "http://localhost:8000/api/v1/data/knowledge/count"
```

#### 清空数据（如需重新导入）

```bash
# 清空意图数据
curl -X DELETE "http://localhost:8000/api/v1/data/intents/clear"

# 清空知识库
curl -X DELETE "http://localhost:8000/api/v1/data/knowledge/clear"
```

## API接口

### 聊天接口

- `POST /api/v1/chat/sessions` - 创建新会话
- `GET /api/v1/chat/sessions` - 获取用户会话列表
- `DELETE /api/v1/chat/sessions/{session_id}` - 删除会话
- `GET /api/v1/chat/sessions/{session_id}/messages` - 获取会话消息
- `POST /api/v1/chat/message` - 发送消息（非流式）
- `POST /api/v1/chat/message/stream` - 发送消息（SSE流式）

### 数据管理接口

- `POST /api/v1/data/intents/import` - 导入意图模板
- `POST /api/v1/data/knowledge/import` - 导入知识库
- `DELETE /api/v1/data/intents/clear` - 清空意图数据
- `DELETE /api/v1/data/knowledge/clear` - 清空知识库

## 数据格式

### 意图模板Excel格式

| 字段 | 说明 | 示例 |
|------|------|------|
| intent_name | 意图名称 | 查询订单 |
| intent_description | 意图描述 | 用户想要查询订单状态 |
| slots | 槽位定义（JSON） | [{"name":"order_id","description":"订单号","required":true}] |
| resolution_method | 解决方式 | function_call / rag / human |
| example_queries | 示例话术（\|分隔） | 查询订单\|我的订单在哪里 |

### 知识库Excel格式

| 字段 | 说明 | 示例 |
|------|------|------|
| category | 知识类别 | 售后政策 |
| topic | 主题 | 退货政策 |
| question | 示例问题 | 购买的商品可以退货吗？ |
| answer | 答案 | 晨星家居支持7天无理由退货... |

## 数据库表结构

### 创建数据库

```sql
CREATE DATABASE IF NOT EXISTS ai_customer_service
DEFAULT CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

USE ai_customer_service;
```

### users表

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id VARCHAR(64) NOT NULL UNIQUE COMMENT '用户唯一标识',
    username VARCHAR(64) NOT NULL UNIQUE COMMENT '用户名',
    password_hash VARCHAR(256) NOT NULL COMMENT '密码哈希',
    nickname VARCHAR(64) NULL COMMENT '昵称',
    email VARCHAR(128) NULL COMMENT '邮箱',
    phone VARCHAR(20) NULL COMMENT '手机号',
    avatar VARCHAR(256) NULL COMMENT '头像URL',
    is_active BOOLEAN DEFAULT TRUE COMMENT '是否激活',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    last_login_at DATETIME NULL COMMENT '最后登录时间',
    INDEX idx_user_id (user_id),
    INDEX idx_username (username)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='用户表';
```

## Agent运行流程

```
用户提问
    │
    ▼
意图识别与槽位提取
    │
    ├─ 意图明确且槽位完整
    │       │
    │       ▼
    │   路由到解决方式
    │       │
    │       ├─ RAG知识检索
    │       ├─ 工具调用(Function Call)
    │       └─ 转人工处理
    │       │
    │       ▼
    │   生成回答 → 流式输出
    │
    └─ 意图不明确或槽位缺失
            │
            ▼
        生成澄清问题
            │
            ▼
        等待用户回复（多轮对话）
```

## 日志

日志文件保存在 `backend/logs/` 目录：
- `app_YYYY-MM-DD.log` - 通用日志
- `agent_YYYY-MM-DD.log` - Agent运行日志
- `error_YYYY-MM-DD.log` - 错误日志

## 扩展开发

### 添加新的业务工具

在 `backend/app/tools/business_tools.py` 中添加新工具：

```python
from langchain_core.tools import tool

@tool("tool_name")
def new_tool(param: str) -> dict:
    """工具描述"""
    # 实现逻辑
    return {"success": True, "data": {...}}
```

然后将工具添加到 `ALL_TOOLS` 列表中。

### 添加新的意图

通过Excel导入或直接调用API添加新的意图模板。

## License

MIT
