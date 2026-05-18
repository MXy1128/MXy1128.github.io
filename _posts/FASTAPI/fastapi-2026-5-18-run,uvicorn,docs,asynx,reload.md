# Day 5 学习总结 —— FastAPI 入门与开发环境配置

## 今日学习内容

今天正式开始学习 Python Web 框架 —— FastAPI，并重新配置了 PyCharm 开发环境。\
重点学习了 FastAPI 的基础运行方式、异步机制以及自动生成 API 文档等核心概念。

***

# 一、FastAPI 是什么

FastAPI 是一个基于 Python 的现代化 Web 框架，主要用于：

- 开发 Web API
- 构建 AI Agent 后端
- 搭建大模型服务接口
- 开发高性能异步服务

FastAPI 的特点：

- 开发速度快
- 自动生成接口文档
- 支持异步
- 类型提示友好
- 性能高（基于 ASGI）

目前很多 AI Agent 项目、RAG 项目、LangChain 服务都会使用 FastAPI 作为后端接口框架。

***

# 二、FastAPI 基础实例

## 1. 创建 FastAPI 实例

FastAPI 的核心入口是创建一个 `FastAPI()` 对象：

```python
from fastapi import FastAPI

app = FastAPI()
```

这里：

- `FastAPI()`：创建 Web 应用
- `app`：整个服务对象
- 后续所有接口都会注册到 app 上

***

## 2. 编写接口

示例：

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def root():
    return {"message": "Hello FastAPI"}
```

这里学习到了：

### `@app.get("/")`

这是一个路由装饰器：

- `get`：HTTP GET 请求
- `/`：访问路径
- 当用户访问网站根路径时，会执行下面的函数

***

## 3. 返回 JSON 数据

FastAPI 默认会把 Python 字典自动转换为 JSON：

```python
return {"message": "Hello FastAPI"}
```

最终浏览器会看到：

```json
{
    "message": "Hello FastAPI"
}
```

这一点非常适合 AI 接口开发。

***

# 三、异步（async）学习

今天学习了 FastAPI 的一个核心能力：

# async 异步

示例：

```python
@app.get("/")
async def root():
    return {"message": "Hello"}
```

***

## 1. 什么是异步

传统程序执行方式：

- 一个任务执行完
- 才能执行下一个任务

异步程序：

- 遇到等待操作时（数据库、网络请求）
- 可以先去执行别的任务
- 提高服务器并发能力

***

## 2. 为什么 FastAPI 强调异步

AI Agent、RAG、LLM 服务经常需要：

- 调用大模型 API
- 访问数据库
- 请求向量数据库
- 网络搜索
- 文件读取

这些都属于：

# IO 操作

IO 操作最大的问题：

- 等待时间长
- CPU 实际没干活

异步可以在等待期间继续处理其他请求。

***

## 3. async 与 await

异步通常配合：

```python
async
await
```

例如：

```python
async def test():
    await some_task()
```

含义：

- `async`：声明异步函数
- `await`：等待异步任务完成

***

## 4. FastAPI 为什么性能高

FastAPI 基于：

- ASGI（异步服务器网关接口）

而不是传统的：

- WSGI

因此：

- 更适合高并发
- 更适合 AI 服务
- 更适合实时接口

***

# 四、FastAPI 的运行方式

今天学习了两种运行方式。

***

# 1. 直接 Run

直接点击 PyCharm 的 Run：

```python
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app)
```

特点：

- 适合简单测试
- 开发阶段方便

***

# 2. 使用 uvicorn 启动

命令：

```bash
uvicorn main:app --reload
```

这是 FastAPI 最常见的启动方式。

***

## 命令详解

### `uvicorn`

ASGI 服务器。

作用：

- 启动 FastAPI 项目
- 处理 HTTP 请求

类似于：

- Flask 的 Werkzeug
- Java 的 Tomcat

***

### `main:app`

含义：

```bash
文件名:FastAPI实例对象
```

例如：

```python
# main.py
app = FastAPI()
```

对应：

```bash
main:app
```

***

### `--reload`

作用：

# 热重载（自动重启）

修改代码后：

- 自动重启服务器
- 不需要手动停止再运行

非常适合开发阶段。

***

## 为什么开发时一定常用 reload

因为频繁修改代码：

- 接口
- 参数
- Agent逻辑
- Prompt

如果每次都手动重启：

- 非常低效

所以：

```bash
uvicorn main:app --reload
```

几乎是 FastAPI 开发标配。

***

# 五、FastAPI 的交互式文档

今天学习了 FastAPI 最强大的功能之一：

# 自动生成交互式 API 文档

***

## 1. 什么是交互式文档

传统后端：

- 需要手写 API 文档

而 FastAPI：

- 自动生成接口文档
- 自动展示参数
- 自动展示返回值
- 可以直接在线测试接口

因此叫：

# 交互式文档

因为：

- 不只是“看”
- 还能直接“调用接口”

***

## 2. 访问方式

启动项目后：

```bash
http://127.0.0.1:8000/docs
```

即可看到：

# Swagger UI 文档页面

***

## 3. docs 页面能做什么

可以：

- 查看所有接口
- 查看请求参数
- 查看返回结构
- 在线发送请求
- 测试 API

这对：

- AI Agent
- 前后端联调
- 接口调试

非常方便。

***

## 4. FastAPI 为什么能自动生成文档

因为：

FastAPI 大量使用：

- Python 类型注解
- Pydantic 数据校验

例如：

```python
def test(name: str, age: int):
```

FastAPI 能自动识别：

- 参数类型
- 是否必填
- 返回格式

因此可以自动生成文档。

***

## 5. 除了 /docs 还有什么

还有：

```bash
/redoc
```

这是另一种风格的 API 文档页面。

特点：

- 更偏阅读
- 更像正式接口文档

***

# 六、今天学习内容的整体意义

今天实际上已经进入：

# AI Agent 后端开发阶段

因为未来很多项目：

- LangChain
- RAG
- MCP
- Agent系统
- 大模型服务

最终都需要：

# 用 FastAPI 暴露接口

例如：

```text
用户请求
   ↓
FastAPI 接口
   ↓
Agent 调度
   ↓
调用大模型
   ↓
返回结果
```

所以 FastAPI 是 AI 工程化的重要基础。

***

# 七、今日核心知识总结

## 已掌握内容

### FastAPI 基础

- FastAPI 框架作用
- app 实例创建
- 路由定义
- JSON 返回

***

### 异步

- async
- await
- IO 操作
- 高并发思想

***

### 服务运行

- PyCharm Run
- uvicorn 启动
- main:app
- \--reload 热更新

***

### API 文档

- Swagger UI
- /docs
- /redoc
- 在线调试接口

***

# 八、下一步建议学习内容

建议下一步学习：

## FastAPI 核心进阶

### 1. 请求参数

学习：

```python
Query
Path
Body
```

***

### 2. Pydantic 数据模型

这是 FastAPI 核心。

***

### 3. POST 请求

真正的 AI 接口几乎都用 POST。

***

### 4. 请求体校验

学习：

```python
BaseModel
```

***

### 5. 文件上传

后续 AI 私厨、多模态 Agent 必学。

***

### 6. 接入大模型

例如：

- OpenAI API
- DeepSeek API
- Qwen API

***

### 7. LangChain + FastAPI

这是后续 AI Agent 核心方向。

***

# 今日学习评价

今天已经从：

- “了解 Agent 概念”

正式进入：

# “AI 工程开发”

阶段。

目前路线已经逐渐清晰：

```text
Python
→ FastAPI
→ LangChain
→ Agent
→ RAG
→ MCP
→ AI工程化
```

这是当前 AI 应用开发非常主流的一条路线。
