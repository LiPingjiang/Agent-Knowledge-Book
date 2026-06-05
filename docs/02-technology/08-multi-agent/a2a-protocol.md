---
title: "A2A 协议 (Agent-to-Agent Protocol)"
description: "深入解析 Google 提出的 A2A 开放协议，涵盖 Agent Card、任务生命周期、发现机制、交互模式及与 MCP 的互补关系"
tags: ["A2A", "protocol", "interoperability", "multi-agent", "Google"]
date: 2025-06-01
author: "Agent Knowledge Book"
---

# A2A 协议 (Agent-to-Agent Protocol)

## 概述

A2A（Agent-to-Agent）是 Google 于 2025 年提出的开放通信协议，旨在解决一个日益突出的问题：不同框架构建的 Agent 无法互相通信。

当前的 Agent 生态呈现碎片化格局。Google ADK、LangGraph、CrewAI、AutoGen 各自构建了封闭的通信机制——ADK Agent 无法直接调度 LangGraph Agent，CrewAI 的 Crew 也无法委派任务给 AutoGen 的 AssistantAgent。这种生态割裂迫使开发者在框架选择上做出非此即彼的决策，或者投入大量精力编写定制化的桥接代码。

A2A 协议的定位类似于 HTTP 对 Web 服务的作用：它不关心 Agent 内部用什么框架实现，只定义 Agent 之间如何发现彼此、如何交换任务、如何报告进度。任何遵循 A2A 规范的 Agent——无论底层是 Python、Java 还是 Go，无论用了哪个 LLM——都可以无缝协作。

## 技术架构

### 传输与线格式

A2A 协议基于 **JSON-RPC 2.0 over HTTP(S)**。与 REST 风格不同，A2A 不使用多路径 URL——所有通信通过单一端点（Agent 的 `url` 字段指定的地址）的不同 JSON-RPC method 完成：

```http
POST https://agents.example.com/code-review HTTP/1.1
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJSUzI1NiIs...

{
  "jsonrpc": "2.0",
  "id": "req-001",
  "method": "message/send",
  "params": {
    "message": {
      "role": "user",
      "parts": [{"kind": "text", "text": "Review this PR for security issues"}],
      "messageId": "msg-uuid-1"
    },
    "configuration": {
      "acceptedOutputModes": ["text/plain", "application/json"],
      "blocking": true
    }
  }
}
```

响应格式也是标准 JSON-RPC 2.0。对于流式响应（`message/stream`），Server 返回 `Content-Type: text/event-stream`，每个 SSE event 的 `data` 字段包含一个 JSON-RPC response 对象。

### JSON-RPC 方法列表（A2A v1.0）

| 方法 | 描述 | 典型用途 |
|------|------|---------|
| `message/send` | 发送消息，同步等待最终结果 | 短时任务（几秒内完成） |
| `message/stream` | 发送消息，通过 SSE 流接收增量更新 | 长时任务、需要进度反馈 |
| `tasks/get` | 查询已有 Task 状态和结果 | 异步轮询 |
| `tasks/list` | 列出 Tasks（支持按 contextId 过滤、分页） | 任务管理 UI |
| `tasks/cancel` | 取消正在执行的 Task | 超时/用户中止 |
| `tasks/resubscribe` | 重新订阅已有 Task 的 SSE 流 | 断线重连 |
| `tasks/pushNotificationConfig/set` | 注册 Webhook 推送通知 | 分布式系统 |
| `tasks/pushNotificationConfig/get` | 获取 Task 的推送配置 | 配置管理 |
| `tasks/pushNotificationConfig/list` | 列出所有推送配置 | 配置管理 |
| `tasks/pushNotificationConfig/delete` | 删除推送配置 | 清理 |
| `agent/authenticatedExtendedCard` | 获取认证后的扩展 Agent Card | 私有能力发现 |

### SSE 流式响应格式

当 Client 调用 `message/stream` 时，Server 返回 W3C 标准的 Server-Sent Events 流：

```
HTTP/1.1 200 OK
Content-Type: text/event-stream

event: message
data: {"jsonrpc":"2.0","id":"req-1","result":{"taskId":"task-abc","contextId":"ctx-1","status":{"state":"working","message":{"role":"agent","parts":[{"kind":"text","text":"正在分析代码..."}]}},"final":false}}

event: message
data: {"jsonrpc":"2.0","id":"req-1","result":{"taskId":"task-abc","artifact":{"artifactId":"art-1","parts":[{"kind":"text","text":"发现 3 个安全问题..."}]},"final":false}}

event: message
data: {"jsonrpc":"2.0","id":"req-1","result":{"taskId":"task-abc","status":{"state":"completed"},"final":true}}
```

流中有两种核心事件类型：**TaskStatusUpdateEvent**（状态变更）和 **TaskArtifactUpdateEvent**（增量产出物）。流以 `"final": true` 标记的事件结束。`tasks/resubscribe` 允许断线重连后继续接收已有 Task 的更新。

### 认证机制

A2A 通过 Agent Card 的 `securitySchemes` 声明支持的认证方案：

```json
{
  "securitySchemes": {
    "oauth2": {
      "type": "oauth2",
      "flows": {
        "clientCredentials": {
          "tokenUrl": "https://auth.example.com/oauth/token",
          "scopes": { "agent:execute": "Execute agent tasks" }
        },
        "authorizationCode": {
          "authorizationUrl": "https://auth.example.com/authorize",
          "tokenUrl": "https://auth.example.com/token",
          "scopes": { "agent:execute": "Execute agent tasks" }
        }
      }
    },
    "apiKey": {
      "type": "apiKey",
      "in": "header",
      "name": "X-API-Key"
    },
    "mutualTls": {
      "type": "mutualTLS"
    }
  },
  "security": [{"oauth2": ["agent:execute"]}]
}
```

支持的认证类型包括 API Key、HTTP Auth（Bearer/Basic）、OAuth 2.0（Authorization Code / Client Credentials / Device Code）、OpenID Connect 和 Mutual TLS。传输层强制要求 TLS 1.2+。

### Push Notification（Webhook）注册

对于长时间运行的任务，Client 无需保持 SSE 连接，而是注册 Webhook 回调：

```json
// Client → Server: 注册 Webhook
{
  "method": "tasks/pushNotificationConfig/set",
  "params": {
    "taskId": "task-abc",
    "pushNotificationConfig": {
      "url": "https://client.example.com/webhooks/a2a",
      "token": "client-verification-token-xyz",
      "authentication": {
        "schemes": ["bearer"],
        "credentials": "server-to-client-token"
      }
    }
  }
}
```

Server 在 Task 状态变更时向 Webhook URL 发送 HTTP POST，附带 `X-A2A-Notification-Token` header 用于客户端验证来源。

### 版本协商

A2A v1.0 通过 Agent Card 的 `protocolVersion` 字段声明协议版本。当前版本经历了 v0.2.1 → v0.3.0 → v1.0 的演进，v1.0 引入了命名空间化的方法名（从 `sendTask` 改为 `message/send`）和正式的版本协商机制。

---

## 核心概念

A2A 协议围绕三个基本原语构建：

**Agent Card** 是 Agent 的身份文件，以 JSON 格式声明 Agent 的能力、技能、通信端点和认证要求。它类似于 API 世界中的 OpenAPI 规范——消费方通过读取 Agent Card 即可了解如何与该 Agent 交互。

**Task** 是 A2A 中的基本工作单元。一个 Task 代表一次从发起到完结的完整交互，包含状态流转（submitted → working → completed/failed）、输入输出 artifacts、以及关联的消息历史。

**Message / Part** 是通信载荷的结构。一条 Message 由一个或多个 Part 组成，每个 Part 可以是文本（TextPart）、文件引用（FilePart）或结构化数据（DataPart）。这种设计允许 Agent 在单条消息中混合传递不同类型的内容。

```mermaid
graph LR
    subgraph "A2A 核心原语"
        AC[Agent Card] -->|声明能力| T[Task]
        T -->|包含| M[Message]
        M -->|由多个| P[Part] -->|组成| M
        T -->|产出| AR[Artifact]
    end
    
    subgraph "Task 状态流转"
        S[submitted] --> W[working]
        W --> C[completed]
        W --> F[failed]
        W --> I[input-required]
        I --> W
    end
```

## Agent Card Schema

Agent Card 是 A2A 协议的入口点。以下是一个完整的 Agent Card 示例：

```json
{
  "name": "CodeReviewAgent",
  "description": "Performs automated code review with security and quality analysis",
  "url": "https://agents.example.com/code-review",
  "version": "1.2.0",
  "provider": {
    "organization": "Example Corp",
    "url": "https://example.com"
  },
  "capabilities": {
    "streaming": true,
    "pushNotifications": true,
    "stateTransitionHistory": true
  },
  "authentication": {
    "schemes": ["OAuth2"],
    "credentials": "Bearer token via Authorization header"
  },
  "defaultInputModes": ["text/plain", "application/json"],
  "defaultOutputModes": ["text/plain", "application/json"],
  "skills": [
    {
      "id": "security-scan",
      "name": "Security Vulnerability Scan",
      "description": "Scans code for common security vulnerabilities (OWASP Top 10)",
      "tags": ["security", "vulnerability", "OWASP"],
      "examples": [
        "Scan this Python file for SQL injection risks",
        "Check authentication logic for vulnerabilities"
      ]
    },
    {
      "id": "quality-review",
      "name": "Code Quality Review",
      "description": "Reviews code for maintainability, readability, and best practices",
      "tags": ["quality", "review", "best-practices"]
    }
  ]
}
```

关键字段说明：

| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | string | Agent 的唯一标识名称 |
| `url` | string | Agent 的通信端点 URL |
| `capabilities` | object | 声明支持的交互模式（streaming、pushNotifications 等） |
| `authentication` | object | 认证方案和凭证传递方式 |
| `skills` | array | Agent 具备的技能列表，每项包含 id、描述和示例 |
| `defaultInputModes` | array | 接受的输入 MIME 类型 |
| `defaultOutputModes` | array | 输出的 MIME 类型 |

## 发现机制 (Discovery)

Client Agent 在发送任务之前，需要先发现并了解 Server Agent 的能力。A2A 定义了三种发现机制：

```mermaid
sequenceDiagram
    participant C as Client Agent
    participant WK as Well-Known URI
    participant R as Registry
    participant S as Server Agent

    Note over C,S: 方式一：Well-Known URI
    C->>WK: GET /.well-known/agent.json
    WK-->>C: 返回 Agent Card

    Note over C,S: 方式二：Curated Registry
    C->>R: 查询 "code review" 能力
    R-->>C: 返回匹配的 Agent Card 列表

    Note over C,S: 方式三：Direct Configuration
    C->>S: GET /agent-card (预配置 URL)
    S-->>C: 返回 Agent Card
```

三种发现机制的对比：

| 机制 | 适用场景 | 优势 | 局限 |
|------|---------|------|------|
| Well-Known URI | 公开 Agent、已知域名 | 零配置，标准化路径 | 需要 DNS 域名 |
| Curated Registry | 企业内部、平台市场 | 支持能力搜索、分类筛选 | 需要维护注册中心 |
| Direct Configuration | 开发测试、私有部署 | 最灵活，无依赖 | 不可扩展，手动维护 |

Well-Known URI 是最简单的发现方式：任何符合 A2A 规范的 Agent 都应在 `/.well-known/agent.json` 路径下暴露自己的 Agent Card。这与 OAuth 的 `/.well-known/openid-configuration` 设计理念一致。

## 四种交互模式

A2A 协议支持四种交互模式，覆盖从简单到复杂的各类场景：

### 同步请求/响应

最简单的模式，Client 发送任务后阻塞等待直到 Server 返回最终结果。适合执行时间短（几秒内完成）的任务。

### 异步轮询

Client 发送任务后获得 Task ID，之后定期轮询任务状态。适合执行时间较长但 Client 不支持接收推送的场景。

### 流式更新 (SSE)

Server 通过 Server-Sent Events 实时推送任务进度和中间结果。Client 维持一个长连接，持续接收状态更新。适合需要实时反馈的交互式场景。

### 推送通知 (Webhooks)

Client 在发送任务时提供回调 URL，Server 在状态变更时主动推送通知到该 URL。适合分布式系统中 Client 无法维持长连接的场景。

```mermaid
sequenceDiagram
    participant C as Client Agent
    participant S as Server Agent

    Note over C,S: 模式一：同步请求/响应
    C->>S: POST /tasks/send {message, skill}
    S-->>C: {status: "completed", artifacts: [...]}

    Note over C,S: 模式二：异步轮询
    C->>S: POST /tasks/send {message}
    S-->>C: {taskId: "abc", status: "working"}
    loop 轮询
        C->>S: GET /tasks/abc
        S-->>C: {status: "working", progress: 60%}
    end
    C->>S: GET /tasks/abc
    S-->>C: {status: "completed", artifacts: [...]}

    Note over C,S: 模式三：SSE 流式
    C->>S: POST /tasks/sendSubscribe {message}
    S-->>C: event: status-update (working)
    S-->>C: event: artifact-update (partial)
    S-->>C: event: status-update (completed)

    Note over C,S: 模式四：推送通知
    C->>S: POST /tasks/send {message, pushNotification: {url}}
    S-->>C: {taskId: "abc", status: "working"}
    S->>C: POST callback_url {status: "completed"}
```

## A2A vs MCP 对比

A2A 和 MCP（Model Context Protocol）经常被混淆，但它们解决的是不同层次的问题：

| 维度 | MCP | A2A |
|------|-----|-----|
| 解决的问题 | LLM 如何使用外部工具和数据 | Agent 之间如何协作通信 |
| 通信双方 | LLM ↔ Tool/Resource | Agent ↔ Agent |
| 抽象层级 | 工具调用层（低层） | 任务编排层（高层） |
| 核心原语 | Tool、Resource、Prompt | Task、Agent Card、Message |
| 交互模式 | 函数调用（同步为主） | 任务委派（支持异步、流式） |
| Agent 自主性 | 工具无自主性，被动响应 | 每个 Agent 有自主决策能力 |
| 状态管理 | 无状态（单次调用） | 有状态（Task 生命周期） |
| 典型用例 | 调用搜索 API、读取数据库 | 委派代码审查给专业 Agent |

两者是互补关系而非替代关系：

```mermaid
graph TB
    subgraph "应用层 - A2A"
        OA[Orchestrator Agent] -->|委派任务| RA[Research Agent]
        OA -->|委派任务| CA[Coding Agent]
        RA -->|返回结果| OA
        CA -->|返回结果| OA
    end
    
    subgraph "工具层 - MCP"
        RA -->|调用工具| S[Search API]
        RA -->|读取资源| DB[(Knowledge Base)]
        CA -->|调用工具| G[Git Operations]
        CA -->|读取资源| FS[File System]
    end
    
    style OA fill:#f9f,stroke:#333
    style RA fill:#bbf,stroke:#333
    style CA fill:#bbf,stroke:#333
```

一个典型的生产系统中，Orchestrator Agent 通过 A2A 协议将任务委派给专业 Agent，而每个专业 Agent 内部通过 MCP 调用具体的工具和数据资源。A2A 管理的是"谁做什么"，MCP 管理的是"怎么做"。

## 实现示例

以下示例使用 Python 构建一个基础的 A2A Server 和 Client。

### A2A Server（基于 FastAPI）

```python
from fastapi import FastAPI, Request
from fastapi.responses import StreamingResponse
from pydantic import BaseModel
from enum import Enum
from uuid import uuid4
from datetime import datetime
import asyncio, json

app = FastAPI()

class TaskStatus(str, Enum):
    SUBMITTED = "submitted"
    WORKING = "working"
    COMPLETED = "completed"
    FAILED = "failed"
    INPUT_REQUIRED = "input-required"

class Part(BaseModel):
    type: str  # "text", "file", "data"
    content: str | None = None
    mime_type: str = "text/plain"

class Message(BaseModel):
    role: str  # "user" or "agent"
    parts: list[Part]

class Task(BaseModel):
    id: str
    status: TaskStatus
    messages: list[Message] = []
    artifacts: list[Part] = []

# In-memory store (use Redis/DB in production)
tasks: dict[str, Task] = {}

AGENT_CARD = {
    "name": "SummaryAgent",
    "url": "http://localhost:8000",
    "capabilities": {"streaming": True, "pushNotifications": False},
    "skills": [{"id": "summarize", "name": "Text Summarization"}],
}

@app.get("/.well-known/agent.json")
async def get_agent_card():
    return AGENT_CARD

@app.post("/tasks/send")
async def send_task(request: Request):
    """Synchronous task execution"""
    body = await request.json()
    task = Task(id=str(uuid4()), status=TaskStatus.SUBMITTED,
                messages=[Message(**body["message"])])
    tasks[task.id] = task
    result = await process_task(task)
    return result.model_dump()

@app.post("/tasks/sendSubscribe")
async def send_subscribe(request: Request):
    """SSE streaming task execution"""
    body = await request.json()
    task = Task(id=str(uuid4()), status=TaskStatus.SUBMITTED,
                messages=[Message(**body["message"])])
    tasks[task.id] = task
    return StreamingResponse(stream_task(task), media_type="text/event-stream")

@app.get("/tasks/{task_id}")
async def get_task(task_id: str):
    """Poll task status"""
    if task_id not in tasks:
        return {"error": "Task not found"}, 404
    return tasks[task_id].model_dump()

async def process_task(task: Task) -> Task:
    task.status = TaskStatus.WORKING
    input_text = task.messages[0].parts[0].content
    # Replace with real LLM call
    task.artifacts.append(Part(type="text", content=f"Summary: {input_text[:100]}..."))
    task.status = TaskStatus.COMPLETED
    return task

async def stream_task(task: Task):
    task.status = TaskStatus.WORKING
    yield f"data: {json.dumps({'type': 'status', 'status': 'working'})}\n\n"
    await asyncio.sleep(1)  # Simulate processing
    task.status = TaskStatus.COMPLETED
    task.artifacts.append(Part(type="text", content="Summarized content"))
    yield f"data: {json.dumps({'type': 'completed', 'artifacts': [p.model_dump() for p in task.artifacts]})}\n\n"
```

### A2A Client

```python
import httpx, json
from typing import AsyncIterator

class A2AClient:
    """A2A protocol client for discovering and interacting with remote agents"""
    
    def __init__(self, agent_url: str):
        self.agent_url = agent_url.rstrip("/")
        self.http = httpx.AsyncClient(timeout=30.0)
    
    async def discover(self) -> dict:
        """Fetch Agent Card from well-known URI"""
        resp = await self.http.get(f"{self.agent_url}/.well-known/agent.json")
        resp.raise_for_status()
        return resp.json()
    
    async def send_task(self, text: str) -> dict:
        """Send a task synchronously"""
        payload = {"message": {"role": "user", "parts": [{"type": "text", "content": text}]}}
        resp = await self.http.post(f"{self.agent_url}/tasks/send", json=payload)
        resp.raise_for_status()
        return resp.json()
    
    async def send_task_streaming(self, text: str) -> AsyncIterator[dict]:
        """Send a task with SSE streaming response"""
        payload = {"message": {"role": "user", "parts": [{"type": "text", "content": text}]}}
        async with self.http.stream("POST", f"{self.agent_url}/tasks/sendSubscribe", json=payload) as resp:
            async for line in resp.aiter_lines():
                if line.startswith("data: "):
                    yield json.loads(line[6:])

# --- Usage ---
async def main():
    client = A2AClient("http://localhost:8000")
    card = await client.discover()
    print(f"Connected to: {card['name']}, skills: {[s['name'] for s in card['skills']]}")
    
    async for event in client.send_task_streaming("Summarize the Q4 performance report..."):
        print(event)
```

## 生产部署考量

**安全认证**：A2A 规范通过 Agent Card 的 `authentication` 字段声明认证方案，推荐 OAuth 2.0 Bearer Token 或 mTLS。企业内部部署可结合 API Gateway 统一处理认证，Agent 本身只需验证 Gateway 转发的身份信息。

**任务状态持久化**：内存存储仅适合开发测试。生产环境需要将 Task 状态持久化到 Redis（短期任务）或数据库（需审计追溯的长期任务），并实现 TTL 机制清理过期任务。

**错误处理与重试**：推荐实现指数退避重试（Exponential Backoff），并在 Task 层面保证幂等——相同的 Task ID 重复提交不应产生副作用。`failed` 状态的 Task 应包含结构化错误信息（error code + message）。

**多租户隔离**：通过 tenant_id 划分任务空间，每个 Client 只能访问自己创建的 Task。结合 Rate Limiting 防止单个 Client 耗尽服务端资源。

**可观测性**：在 A2A 消息中透传 trace_id（遵循 W3C Trace Context 标准），将 Agent 间调用链可视化在 Jaeger 或类似平台上，便于排查任务失败和延迟问题。

## 参考

- [Google, 2025] "Agent-to-Agent (A2A) Protocol Specification" — [github.com/google/A2A](https://github.com/google/A2A)
- [Google, 2025] "A2A: A new era of agent interoperability" — Google Cloud Blog
- [Anthropic, 2024] "Model Context Protocol (MCP)" — A2A 的互补协议
- 相关章节：[通信协议概述](./communication-protocols.md)、[编排者-工作者模式](./orchestrator-worker.md)
