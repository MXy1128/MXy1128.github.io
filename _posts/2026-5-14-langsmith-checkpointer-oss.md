# AI 私厨（Personal Chief）项目学习总结

# 一、项目概述

本次完成了一个基于 LangChain + 多模态大模型 的智能体项目 —— AI 私厨（Personal Chief）。

该项目能够：

- 上传冰箱或厨房食材图片
- 自动识别图片中的食材
- 调用 Web 搜索工具搜索菜谱
- 根据营养与难度进行智能排序
- 输出结构化食谱推荐
- 支持上下文记忆与连续对话
- 支持 FastAPI 服务化部署
- 支持 OSS 图片上传
- 支持 LangSmith 调试与监控

整个项目已经具备：

```text
多模态输入 + Agent + Tool + Memory + API + 前后端联调
```

的完整 AI 工程化能力。

---

# 二、项目核心功能

## 1. 多模态图片识别

使用支持多模态能力的大模型：

- qwen3.5-plus
- qwen3-omni-flash

实现：

- 图片输入
- 文本输入
- 图片 + 文本联合输入

核心代码：

```python
from langchain.messages import HumanMessage

multimodal_message = HumanMessage(
    content=[
        {"type": "image", "url": image_url},
        {"type": "text", "text": "帮我看看这些食材能做些什么？"}
    ]
)
```

理解了：

```text
多模态消息本质上是结构化 Message。
```

模型会：

- 自动识别食材
- 判断食材种类
- 分析食材状态
- 生成可用食材列表

---

## 2. Agent 智能体开发

通过：

```python
from langchain.agents import create_agent
```

创建 Agent：

```python
agent = create_agent(
    model=model,
    tools=[web_search],
    system_prompt=system_prompt,
    checkpointer=checkpointer
)
```

理解了 Agent 的核心结构：

| 组件 | 作用 |
|---|---|
| Model | 大模型推理 |
| Tool | 外部工具调用 |
| Prompt | 行为规则 |
| Memory | 上下文记忆 |
| Agent | 工作流调度 |

核心理解：

```text
Agent = Model + Tool + Prompt + Memory
```

---

# 三、Prompt 工程

系统提示词中定义了 Agent 的完整工作流程：

```text
1. 识别食材
2. 评估新鲜度
3. 调用 web_search
4. 搜索菜谱
5. 多维评分
6. 结构化输出
```

重点理解：

```text
Prompt 不只是“提示词”
而是在定义 Agent 的行为逻辑。
```

---

# 四、Tool 工具调用机制

## 1. Tavily Web 搜索工具

使用：

```python
from langchain_tavily import TavilySearch
```

初始化：

```python
web_search = TavilySearch(
    max_results=5,
    topic="general"
)
```

Agent 会自动：

- 分析用户需求
- 生成搜索关键词
- 调用搜索工具
- 获取互联网结果
- 基于结果继续推理

实现了：

```text
模型 + 外部知识
```

的结合。

---

## 2. Tool Calling 工作流程

真正理解了 Agent 的运行机制：

```text
用户输入
    ↓
Agent分析任务
    ↓
决定是否调用Tool
    ↓
Tool返回结果
    ↓
模型继续推理
    ↓
生成最终回答
```

这是现代 AI Agent 的核心能力。

---

# 五、重点学习：Checkpointer 记忆系统

这是本次项目最核心的内容之一。

---

## 1. 为什么需要 Memory

普通大模型：

```text
每次请求都是独立的
```

不会自动记忆历史对话。

因此：

```text
用户：
我喜欢第1道菜

模型：
不知道“第1道菜”是什么
```

而 Agent 想实现：

```text
连续聊天
上下文理解
历史记忆
```

就必须引入：

# Memory（记忆系统）

---

# 六、Checkpointer 深入理解

## 1. Checkpointer 的本质

LangGraph 中：

```python
from langgraph.checkpoint.sqlite import SqliteSaver
```

Checkpointer 的作用：

```text
保存 Agent 的运行状态
```

包括：

- 历史消息
- Tool 调用记录
- Agent 状态
- 上下文数据

本质上：

```text
Checkpointer = Agent 状态持久化系统
```

---

## 2. SqliteSaver

本项目使用：

```python
checkpointer = SqliteSaver(connection)
```

实现：

```python
import sqlite3

connection = sqlite3.connect(
    "db/personal_chief.db",
    check_same_thread=False
)

checkpointer = SqliteSaver(connection)
checkpointer.setup()
```

作用：

- 自动创建数据库表
- 自动保存会话
- 自动恢复上下文

---

## 3. thread_id 机制

核心代码：

```python
config = {
    "configurable": {
        "thread_id": "6"
    }
}
```

理解：

```text
thread_id = 会话ID
```

不同 thread_id：

- 对应不同用户
- 对应不同聊天
- 对应不同上下文

这本质上是：

```text
多会话隔离机制
```

---

## 4. 为什么 LangGraph 能自动记忆

因为：

```text
每次 Agent 调用后：
LangGraph 会自动保存状态
```

下一次调用时：

```text
根据 thread_id 自动恢复状态
```

因此实现：

```text
连续对话
上下文理解
长期聊天
```

---

## 5. 获取历史消息

学习了如何从 Checkpointer 中读取消息：

```python
checkpoint = checkpointer.get(
    {"configurable": {"thread_id": thread_id}}
)
```

然后：

```python
messages = checkpoint["channel_values"]["messages"]
```

理解了：

```text
LangGraph 实际上在维护整个消息状态树
```

---

## 6. 清空会话

实现：

```python
checkpointer.delete_thread(thread_id)
```

作用：

- 删除历史消息
- 清空上下文
- 重置会话状态

---

## 7. 对 Checkpointer 的核心理解

真正理解：

```text
Memory 不是“模型自己记住了”

而是：
框架帮模型管理状态。
```

这是 Agent 工程化最关键的思想之一。

---

# 七、重点学习：LangSmith

本次项目中重点学习了：

# LangSmith

---

## 1. LangSmith 是什么

LangSmith 是：

```text
LangChain 官方 Agent 调试平台
```

作用类似：

- AI 版 APM
- AI 调试平台
- Agent 监控系统

---

## 2. LangSmith 的核心能力

支持：

- Prompt 调试
- Tool 调用监控
- 调用链追踪
- Token 消耗分析
- Agent 状态查看
- 错误定位
- 云端部署

---

## 3. 配置 LangSmith

.env：

```env
LANGSMITH_API_KEY=xxx
LANGSMITH_TRACING=true
LANGSMITH_PROJECT=lc-course
```

重点：

```env
LANGSMITH_TRACING=true
```

开启后：

```text
所有 Agent 调用都会自动上传到 LangSmith
```

---

## 4. LangSmith 调试能力

能够看到：

```text
用户输入
    ↓
Prompt
    ↓
模型调用
    ↓
Tool调用
    ↓
Tool返回
    ↓
最终输出
```

真正实现：

# Agent 全链路可视化

---

## 5. LangSmith 最大价值

传统 AI 开发：

```text
模型为什么这样回答？
完全不知道。
```

而 LangSmith：

```text
可以看到 Agent 每一步在干什么。
```

这是 AI 工程化调试的重要能力。

---

## 6. LangSmith Studio

通过：

```text
https://smith.langchain.com/studio/
```

连接本地 Agent：

```text
http://127.0.0.1:2024
```

实现：

- GUI 调试
- 在线测试
- Tool 分析
- Prompt 调试

---

## 7. 对 LangSmith 的理解

真正理解：

```text
LangSmith = AI Agent 的可观测性平台
```

类似：

- Prometheus
- Grafana
- Jaeger

在 AI Agent 领域的角色。

---

# 八、重点学习：OSS 对象存储

这是多模态工程化中的核心部分。

---

## 1. 为什么不能直接上传 Base64

一开始：

```text
图片 → Base64 → 发送给模型
```

问题：

- Token 占用巨大
- 内存消耗高
- 会话上下文暴涨
- 性能差

因此：

```text
生产环境几乎不会直接传 Base64
```

---

## 2. 正确方案：OSS

正确流程：

```text
前端上传图片
    ↓
OSS存储图片
    ↓
返回图片URL
    ↓
URL发送给模型
```

优势：

| 方案 | 优点 |
|---|---|
| Base64 | 简单 |
| OSS URL | 性能高、Token低、可扩展 |

---

## 3. 阿里云 OSS

学习了：

- Bucket
- RAM权限
- AccessKey
- 跨域配置
- 公共读权限

---

## 4. OSS 上传流程

完整流程：

```text
用户上传图片
    ↓
前端请求OSS签名
    ↓
前端直传OSS
    ↓
OSS返回URL
    ↓
URL发送给Agent
```

重点理解：

```text
文件不经过后端服务器
```

这是大型系统常见架构。

---

## 5. OSS 的工程意义

真正理解：

```text
AI 工程不仅是模型
还包括：
存储
网络
权限
文件服务
```

---

# 九、FastAPI 服务化

## 1. FastAPI 项目结构

```text
app/
├── agents/
├── api/
├── models/
├── common/
├── static/
```

理解了：

- 模块化开发
- API分层
- 数据模型分层

---

## 2. Restful API

实现：

```python
POST /chat/stream
GET /chat/messages
DELETE /chat/messages
```

真正完成：

```text
Agent → API → 前端
```

完整工程闭环。

---

## 3. StreamingResponse

学习：

```python
StreamingResponse(...)
```

实现：

- 边生成边输出
- ChatGPT 风格流式响应

---

# 十、完整系统架构理解

最终完整流程：

```text
用户上传图片
    ↓
前端上传OSS
    ↓
OSS返回URL
    ↓
FastAPI接收请求
    ↓
Agent调用模型
    ↓
Agent调用Tool
    ↓
LangGraph维护状态
    ↓
Checkpointer保存记忆
    ↓
LangSmith记录调用链
    ↓
流式返回结果
```

真正完成了：

# 一个完整的 AI Agent 工程系统

---

# 十一、核心技术栈

## AI / Agent

- LangChain
- LangGraph
- LangSmith
- Tavily
- Qwen 多模态模型

---

## 后端

- FastAPI
- Uvicorn
- StreamingResponse

---

## 数据与存储

- SQLite
- Checkpointer
- 阿里云 OSS

---

## Python生态

- dotenv
- pillow
- pydantic

---

# 十二、核心收获

## 1. 理解了 Agent 的真正结构

```text
Agent ≠ 大模型

Agent = 模型 + Tool + 状态 + 工作流
```

---

## 2. 理解了 Memory 的本质

```text
记忆不是模型自己记住

而是：
框架在维护状态。
```

---

## 3. 理解了 AI 工程化

真正接触到了：

- 多模态系统
- Agent 架构
- 状态管理
- 服务化部署
- 文件存储
- 可观测性平台
- 流式交互

已经不仅仅是：

```text
调用大模型 API
```

而是在：

# 开发完整的 AI 应用系统