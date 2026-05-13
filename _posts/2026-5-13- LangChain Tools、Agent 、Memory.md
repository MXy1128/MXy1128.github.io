# Day3 学习笔记 —— LangChain Tools、Agent 与 Memory 工程化

> 今日学习方向：
>
> 从“大模型调用”正式进入：
>
> # AI Agent 工程化开发

今天的学习已经不仅仅是：

```text
调用一次大模型 → 返回一次结果
```

而是开始学习：

```text
如何让 AI：
- 调用工具
- 自动推理
- 联网搜索
- 管理状态
- 保存记忆
- 长期运行
```

这意味着已经正式进入：

# AI Agent 开发阶段

---

# 第一部分：LangChain Tools 与结构化输出

---

# 一、今日学习内容概览

今天主要学习了：

- LangChain Tool 工具机制
- Tavily 搜索工具
- Agent 与 Tool 调用流程
- 自定义 Tool 封装
- `@tool` 装饰器
- Tool Description 的本质
- Structured Output（结构化输出）
- Pydantic 数据模型
- BaseModel 与 Field
- AI 输出约束
- 嵌套结构化数据

---

# 二、什么是 Tool

## 1. Tool 的本质

Tool 本质上是：

> 给大模型提供“外部能力”。

因为大模型本身：

- 不能联网
- 不能访问数据库
- 不能调用 API
- 不能执行真实代码
- 不能获取实时信息

所以：

> Tool = AI 与现实世界交互的桥梁

---

## 2. Tool 可以做什么

| Tool 类型 | 功能 |
|---|---|
| Web Search | 联网搜索 |
| Python Tool | 执行 Python |
| SQL Tool | 查询数据库 |
| Retriever | 查询知识库 |
| API Tool | 调用外部接口 |

---

# 三、Agent 的核心思想

## Agent 是什么

Agent 是：

> 可以“自动决定调用什么工具”的智能体。

---

## Agent 的运行流程

```text
用户问题
   ↓
LLM分析任务
   ↓
判断是否需要Tool
   ↓
调用Tool
   ↓
获得结果
   ↓
继续推理
   ↓
输出最终答案
```

---

# 四、ReAct 思想

很多 Agent 使用：

# ReAct（Reason + Act）

即：

| 阶段 | 含义 |
|---|---|
| Reason | 推理 |
| Act | 行动（调用工具） |

---

## ReAct 工作流程

```text
思考
↓
决定调用工具
↓
获取工具结果
↓
继续思考
↓
输出答案
```

---

# 五、Tavily 搜索工具

## 1. Tavily 是什么

Tavily 是专门为 AI Agent 提供的搜索引擎。

相比传统搜索：

- 更适合大模型
- 搜索结果更干净
- 支持摘要
- 更适合 Agent / RAG

---

# 六、Tavily 基本使用

## 1. 导包

```python
from langchain_tavily import TavilySearch
```

---

## 2. 创建搜索工具

```python
search_tool = TavilySearch(
    max_results=5,
    topic="general"
)
```

---

# 七、Tavily 参数详解

---

## 1. max_results

```python
max_results=5
```

含义：

> 最多返回 5 条搜索结果。

类似：

```text
搜索结果 Top5
```

---

## 2. topic

```python
topic="general"
```

表示搜索类型。

---

### 常见类型

| 参数 | 含义 |
|---|---|
| general | 普通搜索 |
| news | 新闻搜索 |
| finance | 金融搜索 |

---

# 八、Tavily 高级参数

---

## include_answer

```python
include_answer=True
```

是否直接返回搜索摘要。

---

## include_raw_content

```python
include_raw_content=True
```

是否返回网页原文。

适合：

- RAG
- 文档分析
- 长文本处理

---

## include_images

```python
include_images=True
```

是否返回图片。

---

## include_image_descriptions

```python
include_image_descriptions=True
```

是否返回图片描述。

---

## search_depth

```python
search_depth="advanced"
```

搜索深度：

| 参数 | 含义 |
|---|---|
| basic | 快速搜索 |
| advanced | 深度搜索 |

---

## time_range

```python
time_range="day"
```

搜索时间范围：

| 参数 | 含义 |
|---|---|
| day | 最近一天 |
| week | 最近一周 |
| month | 最近一个月 |

---

## include_domains

```python
include_domains=["github.com"]
```

只搜索指定网站。

---

## exclude_domains

```python
exclude_domains=["zhihu.com"]
```

排除某些网站。

---

# 九、自定义 Tool

## 为什么需要自定义 Tool

企业开发中：

官方 Tool 不可能覆盖所有需求。

因此需要：

- 查询内部系统
- 调企业数据库
- 调公司 API
- 查询日志
- 调业务服务

---

# 十、@tool 装饰器

LangChain 提供：

```python
@tool
```

可以快速把函数转换成 Tool。

---

## 示例

```python
from langchain.tools import tool

@tool
def add(a:int,b:int):
    """Add two numbers"""
    return a+b
```

---

# 十一、docstring 的真正作用（重点）

```python
"""Add two numbers"""
```

不是普通注释。

而是：

# 给大模型看的 Tool Description

---

# 十二、Tool Description 的本质

Agent 会把 Tool 信息发给大模型：

```text
Tool Name: add
Description: Add two numbers
```

模型根据这些内容决定：

```text
是否调用这个工具
```

---

# 十三、为什么 Tool Description 很重要

因为：

> 大模型是否会正确调用 Tool，
> 完全依赖 description。

---

## 好的 Description

```python
"""Search the web for recent information and news"""
```

模型立刻知道：

- 联网搜索
- 查新闻
- 获取最新信息

---

## 不好的 Description

```python
"""Tool"""
```

模型根本不知道工具用途。

---

# 十四、Tool 的底层结构

例如：

```python
@tool
def web_search(query:str):
    """Search the web for information"""
```

底层实际上类似：

```python
Tool(
    name="web_search",
    description="Search the web for information",
    func=web_search
)
```

---

# 十五、Tool 参数也会给模型看

例如：

```python
def web_search(query:str)
```

模型会知道：

```text
这个工具需要一个字符串参数 query
```

---

# 十六、什么是结构化输出

## 默认问题

大模型默认输出：

```text
张三今年18岁，来自北京
```

这种文本：

- 不稳定
- 不方便程序解析
- 容易格式变化

---

## 因此需要

# Structured Output（结构化输出）

例如：

```json
{
  "name":"张三",
  "age":18,
  "city":"北京"
}
```

---

# 十七、Pydantic 是什么

Pydantic 是 Python 的数据验证与结构化建模库。

---

## 核心作用

| 功能 | 说明 |
|---|---|
| 数据结构定义 | 定义对象结构 |
| 类型校验 | 检查数据类型 |
| 自动转换 | 自动类型转换 |
| JSON Schema | 生成标准结构 |
| 结构化输出 | 约束 AI 输出 |

---

# 十八、BaseModel

## 基础写法

```python
from pydantic import BaseModel

class User(BaseModel):
    name:str
    age:int
```

---

## 含义

表示：

```text
User 是一个结构化数据模型
```

---

# 十九、Field 的作用

## 基础写法

```python
Field(description="...")
```

---

## 作用

| 功能 | 说明 |
|---|---|
| description | 字段描述 |
| 默认值 | 设置默认值 |
| 校验规则 | 数据限制 |
| 示例 | 提供示例 |

---

# 二十、Field(description=...) 的真正意义

例如：

```python
Field(description="The final answer for user")
```

本质上：

# 是给大模型看的字段说明

模型会理解：

```text
这里应该填写最终答案
```

---

# 二十一、结构化输出示例

---

## 定义网页引用结构

```python
class Reference(BaseModel):
    title:str
    url:str
```

---

## 定义最终回答结构

```python
class AnswerInfo(BaseModel):
    answer:str
    reference:list[Reference]
```

---

# 二十二、list[Reference] 的含义

```python
list[Reference]
```

表示：

```text
一个列表
列表里的每项都是 Reference 对象
```

---

# 二十三、最终输出效果

```json
{
  "answer":"GPT-5 是 OpenAI 发布的新模型",
  "reference":[
    {
      "title":"OpenAI官网",
      "url":"https://openai.com"
    },
    {
      "title":"GitHub",
      "url":"https://github.com"
    }
  ]
}
```

---

# 二十四、为什么结构化输出非常重要

因为企业开发中：

AI 输出通常需要：

- 存数据库
- 返回前端
- API 通信
- 工作流流转
- 多 Agent 协作

所以：

> 输出必须稳定、标准化、可解析。

---

# 二十五、这一部分学习的核心本质

今天实际上学习的是：

# “如何让 AI 调用外部能力，并输出标准化结果”

---

# 第二部分：AI Agent Memory（记忆机制）

---

# 二十六、什么是 AI Agent 的记忆（Memory）

## Memory 的本质

Memory 的作用是：

> 让 AI “记住之前发生过的事情”。

因为正常的大模型：

```text
每次调用都是独立的
```

它不会自动记住：

- 上一次对话
- 用户信息
- 历史任务
- 之前执行过什么

所以：

> 需要人为给 Agent 增加“记忆系统”。

---

# 二十七、短期记忆（Short-Term Memory）

## 短期记忆是什么

短期记忆：

> 当前会话中的上下文记忆。

例如：

```text
用户：我叫张三
AI：你好张三

用户：我今年18岁
AI：好的

用户：我叫什么？
AI：你叫张三
```

这里 AI 能回答：

是因为：

> 历史消息被重新拼接进 Prompt 中。

---

# 二十八、长期记忆（Long-Term Memory）

## 长期记忆是什么

长期记忆：

> 跨会话、跨任务的持久化记忆。

例如：

今天：

```text
用户：我喜欢Python
```

明天：

```text
AI：你之前提到你喜欢Python
```

这种能力：

就是长期记忆。

---

# 二十九、为什么 LLM 本身没有记忆

## 核心原因

LLM 本质上：

```text
是“无状态”的
```

即：

> 每次请求都是全新的。

模型不会自动保存：

- 聊天记录
- 用户数据
- 历史状态

---

# 三十、LLM 为什么还能“记住上下文”

因为：

> 开发者把历史消息重新拼接给模型。

例如：

```text
用户：你好
AI：你好

用户：我叫张三
AI：你好张三
```

实际上第三次调用时：

发送给模型的 Prompt 是：

```text
用户：你好
AI：你好

用户：我叫张三
AI：你好张三

用户：我叫什么？
```

所以：

> 不是模型记住了，
> 而是程序重新把历史对话发给了模型。

---

# 三十一、LangGraph 中的 Memory

LangGraph 提供：

# Checkpointer

用于实现：

- 对话记忆
- 状态保存
- Agent 恢复
- 持久化存储

---

# 三十二、Checkpointer 的本质

Checkpointer 本质上是：

> Agent 的状态存储器。

它负责：

- 保存历史消息
- 保存 Agent State
- 保存执行状态
- 恢复历史上下文

---

# 三十三、InMemorySaver

## 基础使用

```python
from langgraph.checkpoint.memory import InMemorySaver
```

---

## 初始化

```python
memory = InMemorySaver()
```

---

## 创建 Agent

```python
agent = create_react_agent(
    model=model,
    tools=tools,
    checkpointer=memory
)
```

---

# 三十四、checkpointer 的作用

```python
checkpointer=memory
```

意思：

> 给 Agent 增加记忆能力。

否则：

每次调用都是全新对话。

---

# 三十五、thread_id 的作用（重点）

## 为什么需要 thread_id

因为：

系统需要知道：

```text
哪些消息属于同一个会话
```

---

## 示例

```python
config = {
    "configurable": {
        "thread_id": "1"
    }
}
```

---

## 含义

```text
thread_id = 当前会话ID
```

同一个 thread_id：

会共享历史记忆。

不同 thread_id：

相互独立。

---

# 三十六、thread_id 的本质

thread_id 本质上类似：

| 系统 | 对应概念 |
|---|---|
| 微信 | 聊天窗口 |
| QQ | 会话 |
| 数据库 | Session |
| Web | 用户会话 |

---

# 三十七、为什么 InMemorySaver 不适合生产环境

因为：

```text
数据只存在内存里
```

程序关闭后：

记忆就没了。

---

# 三十八、内存溢出问题

随着聊天变多：

历史消息越来越多。

最终会导致：

- 内存占用越来越大
- Prompt 越来越长
- Token 成本暴涨
- 上下文超限

所以：

> Memory 必须持久化存储。

---

# 三十九、Memory 持久化存储

LangGraph 提供：

## 数据库存储方案

例如：

| 存储方案 | 说明 |
|---|---|
| SQLite | 轻量本地数据库 |
| Postgres | 企业级数据库 |

---

# 四十、SQLite Checkpointer

## 安装依赖

```bash
uv add langgraph-checkpoint-sqlite
```

---

# 四十一、使用步骤

---

## 1. 导入依赖

```python
from langgraph.checkpoint.sqlite import SqliteSaver
```

---

## 2. 初始化数据库 URL

```python
sqlite_url = "sqlite:///memory.db"
```

---

## 3. 初始化 Checkpointer

```python
memory = SqliteSaver.from_conn_string(sqlite_url)
```

---

## 4. 自动建表

```python
memory.setup()
```

---

## 为什么需要建表

因为：

数据库本质是：

# “表结构”

记忆数据最终会存进：

- message表
- checkpoint表
- state表

所以：

需要自动创建数据库表。

---

# 四十二、初始化 Agent

```python
agent = create_react_agent(
    model=model,
    tools=tools,
    checkpointer=memory
)
```

---

# 四十三、SQLite 的优点

| 优点 | 说明 |
|---|---|
| 轻量 | 本地文件即可 |
| 简单 | 无需部署 |
| 适合学习 | 非常方便 |
| 支持持久化 | 关闭程序数据不丢 |

---

# 四十四、Postgres 的作用

生产环境通常使用：

# Postgres

因为：

- 并发更高
- 更稳定
- 支持分布式
- 更适合企业系统

---

# 四十五、多轮对话的问题

随着聊天越来越长：

历史消息会越来越多。

最终：

# 超出模型上下文窗口限制

---

# 四十六、什么是上下文窗口（Context Window）

Context Window：

> 模型一次最多能看到的 Token 数量。

例如：

| 模型 | Context |
|---|---|
| GPT-3.5 | 16K |
| GPT-4 | 128K |
| Claude | 200K+ |

---

# 四十七、上下文超限的问题

如果历史消息太长：

会导致：

- 超过模型最大输入长度
- Token 成本暴涨
- 推理变慢
- 模型遗忘早期信息

---

# 四十八、LangChain 的 Memory 管理策略

LangChain 提供了一系列：

# Memory 管理机制

用于控制：

- 历史长度
- Token 消耗
- 上下文大小

---

# 四十九、修剪（Trim）

## 修剪是什么

修剪：

> 删除部分历史消息。

---

## 常见方式

### 删除前 n 条

```text
保留最近消息
删除最早消息
```

---

### 删除后 n 条

```text
保留早期上下文
删除最新消息
```

---

# 五十、删除（Delete）

## Delete 的本质

Delete：

> 永久删除 AgentState 快照。

包括：

- 历史消息
- 状态数据
- 记忆信息

---

# 五十一、总结摘要（Summarization）

## 为什么需要总结

因为：

早期消息越来越多。

所以：

> 用“摘要”替代完整历史消息。

---

# 五十二、Summarization 工作流程

```text
旧消息
↓
LLM总结
↓
得到摘要
↓
摘要 + 最近消息
↓
重新组成上下文
```

---

# 五十三、总结摘要的核心思想

例如：

原始消息：

```text
用户聊了100轮
```

总结后：

```text
用户是Python开发者
正在学习AI Agent
喜欢LangChain
```

这样：

可以大幅减少 Token。

---

# 五十四、SummarizationMiddleware

LangChain 提供：

# SummarizationMiddleware

用于：

- 自动总结历史消息
- 自动替换旧消息
- 压缩上下文

---

# 五十五、总结摘要模型

注意：

> Summarization 通常需要额外的 LLM。

因为：

总结本身也是：

# 一次模型推理

---

# 五十六、Memory 的工程化本质

今天学习的核心实际上是：

# “如何管理 Agent 的状态”

包括：

- 对话历史
- 用户状态
- 上下文
- 长期记忆
- 会话恢复

---

# 五十七、Memory 系统的完整架构

```text
用户输入
↓
Agent
↓
Checkpointer
↓
读取历史状态
↓
拼接上下文
↓
LLM推理
↓
保存最新状态
```

---

# 五十八、今日核心技术点总结

| 技术 | 本质 |
|---|---|
| Tool | 给模型增加能力 |
| Agent | 自动调用工具 |
| ReAct | 推理 + 行动 |
| Tavily | AI 搜索引擎 |
| @tool | 自定义工具 |
| docstring | Tool 描述 |
| Structured Output | 结构化输出 |
| Pydantic | 数据模型 |
| BaseModel | 结构定义 |
| Field | 字段说明 |
| Short-Term Memory | 当前会话记忆 |
| Long-Term Memory | 跨任务记忆 |
| Checkpointer | 状态存储器 |
| InMemorySaver | 内存记忆 |
| thread_id | 会话ID |
| SQLiteSaver | 数据库存储 |
| PostgresSaver | 企业级存储 |
| setup() | 自动建表 |
| Context Window | 上下文窗口 |
| Trim | 修剪历史消息 |
| Delete | 删除状态 |
| Summarization | 摘要压缩 |
| SummarizationMiddleware | 自动摘要中间件 |

---

# 五十九、今日学习收获

今天已经从：

```text
简单调用大模型
```

开始进入：

# AI Agent 工程化开发

已经接触到：

- Tool Calling
- Agent 推理
- 联网搜索
- 结构化输出
- Memory 管理
- Agent 状态系统
- 长期记忆
- Context 管理
- 状态持久化

这是从：

# “使用 AI”

进入：

# “开发 AI Agent”

的重要一步。

---

# 六十、后续学习方向

接下来可以继续深入：

- LangGraph
- StateGraph
- 多 Agent 协作
- Workflow
- MCP
- RAG
- 向量数据库
- 用户画像
- 长期记忆架构
- Agent 状态恢复
- Agent 生命周期管理

逐渐形成完整的：

# AI Agent 工程化体系。