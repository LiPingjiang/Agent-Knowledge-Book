# 流式响应与实时通信

Agent 系统的响应延迟通常在数秒到数十秒之间（多步推理 + 工具调用），流式响应（Streaming）是改善用户体验的关键技术手段。本章深入讲解流式响应在 Agent 系统中的协议原理、实现模式和工程实践。

## SSE 协议原理

Server-Sent Events（SSE）是 LLM API 最广泛采用的流式传输协议。它基于 HTTP/1.1 的分块传输编码（chunked transfer encoding），服务器单向推送事件到客户端。

### 协议规范

SSE 的 HTTP 响应头和消息格式：

```http
HTTP/1.1 200 OK
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive
Transfer-Encoding: chunked

data: {"id":"chatcmpl-xxx","choices":[{"delta":{"content":"Hello"}}]}

data: {"id":"chatcmpl-xxx","choices":[{"delta":{"content":" World"}}]}

data: [DONE]
```

核心规范要点：每条消息以 `data: ` 前缀开始，以两个换行符 `\n\n` 结束。支持 `event:`（事件类型）、`id:`（事件 ID，用于断线重连）、`retry:`（重连间隔）等字段。客户端通过 `EventSource` API 或手动解析响应流来消费数据。

与 WebSocket 的关键区别在于 SSE 是单向的（服务器 → 客户端）、基于 HTTP（无需协议升级）、原生支持自动重连。对于 LLM 场景，"请求-流式响应"模式天然适合 SSE。

### 各厂商 Streaming API 实现

**OpenAI Streaming：**

```python
from openai import OpenAI
client = OpenAI()

# 基础流式请求
stream = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "分析这段代码的性能问题"}],
    stream=True
)

full_content = ""
for chunk in stream:
    delta = chunk.choices[0].delta
    if delta.content:
        full_content += delta.content
        print(delta.content, end="", flush=True)
    # 流式中的 tool_call 需要增量拼接
    if delta.tool_calls:
        for tc in delta.tool_calls:
            # tc.function.arguments 是增量片段，需拼接完整 JSON
            pass
```

**Anthropic Streaming（事件驱动模型）：**

Anthropic 采用更细粒度的事件模型，每个事件有明确的类型标识：

```python
import anthropic
client = anthropic.Anthropic()

with client.messages.stream(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[{"role": "user", "content": "设计一个缓存方案"}]
) as stream:
    for event in stream:
        match event.type:
            case "content_block_start":
                # 新的内容块开始（text 或 tool_use）
                if event.content_block.type == "tool_use":
                    print(f"[调用工具: {event.content_block.name}]")
            case "content_block_delta":
                # 增量内容
                if event.delta.type == "text_delta":
                    print(event.delta.text, end="", flush=True)
                elif event.delta.type == "input_json_delta":
                    # 工具调用参数的增量 JSON 片段
                    pass
            case "message_stop":
                print("\n[完成]")
```

Anthropic 的事件模型优势在于：`content_block_start` 预告了内容类型，客户端可以在流式过程中就开始分路处理（文本渲染 vs 工具调用准备），而不必等待整个 block 结束。

## 流式场景下的工具调用

Agent 系统面临的核心挑战：流式输出中穿插工具调用。模型可能在生成文本中途决定调用工具，工具执行完毕后继续生成。这要求 Agent 循环支持"流式中断-工具执行-流式恢复"模式。

### 增量 JSON 拼接

当模型以流式方式输出工具调用参数时，每个 chunk 只包含 JSON 的一个片段。客户端需要维护状态机来正确拼接：

```python
class StreamingToolCallAccumulator:
    """流式工具调用参数的增量拼接器"""
    
    def __init__(self):
        self.tool_calls: dict[int, dict] = {}  # index -> {id, name, arguments_buffer}
    
    def process_delta(self, delta_tool_calls: list) -> list[dict] | None:
        """处理增量片段，返回已完成的工具调用（如有）"""
        completed = []
        for tc_delta in delta_tool_calls:
            idx = tc_delta.index
            if idx not in self.tool_calls:
                self.tool_calls[idx] = {
                    "id": tc_delta.id,
                    "name": tc_delta.function.name or "",
                    "arguments_buffer": ""
                }
            
            if tc_delta.function:
                if tc_delta.function.name:
                    self.tool_calls[idx]["name"] = tc_delta.function.name
                if tc_delta.function.arguments:
                    self.tool_calls[idx]["arguments_buffer"] += tc_delta.function.arguments
        
        return completed
    
    def finalize(self) -> list[dict]:
        """流结束时，解析所有已拼接完成的工具调用"""
        import json
        results = []
        for tc in self.tool_calls.values():
            results.append({
                "id": tc["id"],
                "name": tc["name"],
                "arguments": json.loads(tc["arguments_buffer"])
            })
        return results
```

### Agent 循环的流式集成

完整的流式 Agent 循环需要在流式输出和工具执行之间无缝切换：

```python
import asyncio
from typing import AsyncIterator

class StreamingAgentLoop:
    """支持流式输出的 Agent 主循环"""
    
    async def run(self, user_message: str) -> AsyncIterator[str]:
        """返回一个异步迭代器，调用者可逐 token 消费"""
        messages = [{"role": "user", "content": user_message}]
        
        while True:
            # 流式请求 LLM
            stream = await self.llm.create_stream(messages=messages)
            
            text_buffer = ""
            tool_call_acc = StreamingToolCallAccumulator()
            
            async for chunk in stream:
                # 逐 token yield 给前端渲染
                if chunk.delta.content:
                    text_buffer += chunk.delta.content
                    yield chunk.delta.content
                
                if chunk.delta.tool_calls:
                    tool_call_acc.process_delta(chunk.delta.tool_calls)
            
            # 流结束，检查是否有工具调用
            tool_calls = tool_call_acc.finalize()
            if not tool_calls:
                break  # 无工具调用，对话结束
            
            # 执行工具调用
            yield "\n\n[正在执行操作...]\n"
            messages.append({"role": "assistant", "content": text_buffer, "tool_calls": tool_calls})
            
            for tc in tool_calls:
                result = await self.execute_tool(tc["name"], tc["arguments"])
                messages.append({
                    "role": "tool",
                    "tool_call_id": tc["id"],
                    "content": str(result)
                })
            
            # 工具结果送回 LLM，继续流式生成
            yield "\n"
```

## 流式 + 结构化输出

流式响应与结构化输出的结合是一个工程难点。约束解码模式下，虽然每个 token 都合规，但客户端在流结束前无法获得完整的可解析 JSON。解决方案有两种：

**方案一：流式增量 JSON 解析（Partial JSON Parsing）**

```python
import ijson  # 基于事件的增量 JSON 解析器

class StreamingJsonParser:
    """边流式接收边解析 JSON 字段"""
    
    def __init__(self):
        self.buffer = b""
        self.parser = None
        self.extracted_fields = {}
    
    def feed(self, chunk: str):
        """喂入新的 JSON 片段"""
        self.buffer += chunk.encode()
        # ijson 可以在 JSON 不完整时解析已到达的字段
        # 例如 {"analysis": "xxx", "decision": 还没到
        # 此时 analysis 字段已可用
        try:
            events = ijson.items(io.BytesIO(self.buffer), '')
            for prefix, event, value in ijson.parse(io.BytesIO(self.buffer)):
                if prefix == "analysis" and event == "string":
                    self.extracted_fields["analysis"] = value
                elif prefix == "decision" and event == "string":
                    self.extracted_fields["decision"] = value
        except ijson.IncompleteJSONError:
            pass  # JSON 尚不完整，继续等待
```

**方案二：先流式生成 thinking，再结构化输出 action（推荐）**

将自由文本推理和结构化决策拆分为两次 LLM 调用，或使用支持 `thinking` 的模型：

```python
# Claude 的 extended thinking 模式天然支持这种拆分
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=8000,
    thinking={"type": "enabled", "budget_tokens": 4000},  # thinking 流式输出
    tools=[agent_decision_tool],
    tool_choice={"type": "tool", "name": "decide"},  # action 结构化输出
    messages=[...]
)
# thinking blocks 可流式渲染给用户看推理过程
# tool_use block 在最后以结构化方式输出决策
```

## 部署层面的流式配置

流式响应对基础设施有特殊要求。最常见的故障是中间代理层对响应进行缓冲（buffering），导致客户端体感上变成了非流式响应。

### Nginx 配置要点

```nginx
location /api/agent/ {
    proxy_pass http://agent-backend;
    
    # 禁用响应缓冲——流式输出的关键
    proxy_buffering off;
    proxy_cache off;
    
    # 禁用 gzip 压缩（压缩需要缓冲足够数据才能开始）
    gzip off;
    
    # 长超时（Agent 可能思考 30s+ 再开始输出）
    proxy_read_timeout 300s;
    proxy_send_timeout 300s;
    
    # 保持连接
    proxy_http_version 1.1;
    proxy_set_header Connection "";
    
    # 透传 SSE 相关头
    proxy_set_header Accept "text/event-stream";
    add_header X-Accel-Buffering "no";  # 告知上游代理不要缓冲
}
```

### Kubernetes Ingress 配置

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/proxy-buffering: "off"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "300"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "300"
    nginx.ingress.kubernetes.io/server-snippets: |
      location /api/agent/stream {
        proxy_set_header X-Accel-Buffering "no";
      }
```

### 云负载均衡器注意事项

AWS ALB、GCP Cloud Load Balancer 默认启用响应缓冲。对于 SSE 流式端点，需要确认 idle timeout 设置足够长（推荐 300s+），并且在 target group 层面关闭 deregistration delay 对流式连接的影响。部分 CDN（如 CloudFront）默认不支持 SSE 透传，需要配置 origin response timeout 和禁用 caching 策略。

## 流式中断与恢复

Agent 的流式执行可能因网络中断、客户端主动取消、或超时被切断。生产系统需要实现优雅处理：

```python
class StreamingSession:
    """管理流式会话的中断与恢复"""
    
    def __init__(self, session_id: str):
        self.session_id = session_id
        self.checkpoint_content = ""  # 已成功发送给客户端的内容
        self.pending_tool_calls = []
    
    async def handle_disconnect(self):
        """客户端断开连接时的处理"""
        # 如果正在执行工具调用，不能中断——工具可能有副作用
        if self.pending_tool_calls:
            # 将工具执行结果存入持久化存储，等待客户端重连
            await self.persist_state()
        # 如果只是在生成文本，可以安全中断
    
    async def resume(self) -> AsyncIterator[str]:
        """客户端重连时恢复流式输出"""
        state = await self.load_persisted_state()
        if state:
            # 先发送断连期间已完成的内容
            yield f"[恢复会话，已完成的内容如下]\n{state.completed_content}\n"
            # 从断点继续 Agent 循环
            async for chunk in self.continue_from_checkpoint(state):
                yield chunk
```

## 背压（Backpressure）处理

当客户端消费速度跟不上服务端生成速度时（例如客户端在做复杂的 UI 渲染），需要背压机制避免内存溢出：

```python
import asyncio

class BackpressureAwareStream:
    """带背压控制的流式输出"""
    
    def __init__(self, max_buffer_size: int = 100):
        self.queue: asyncio.Queue = asyncio.Queue(maxsize=max_buffer_size)
        self.cancelled = False
    
    async def producer(self, llm_stream):
        """生产者：从 LLM 流中读取 token"""
        async for chunk in llm_stream:
            if self.cancelled:
                break
            # 如果队列满了，这里会自动等待（背压生效）
            await self.queue.put(chunk)
        await self.queue.put(None)  # 结束信号
    
    async def consumer(self) -> AsyncIterator[str]:
        """消费者：按客户端消费速度输出"""
        while True:
            chunk = await self.queue.get()
            if chunk is None:
                break
            yield chunk.delta.content or ""
    
    def cancel(self):
        """客户端取消时调用"""
        self.cancelled = True
```

## 性能基准参考

| 指标 | 典型值 | 说明 |
|------|--------|------|
| 首 token 延迟（TTFT） | 200ms - 2s | 取决于 prompt 长度和模型 |
| token 间延迟（ITL） | 20-80ms | 取决于模型和负载 |
| 流式 vs 非流式的整体延迟差异 | 几乎无差异 | 总 token 数相同，只是交付方式不同 |
| SSE 连接建立开销 | < 50ms | 复用 HTTP 连接时更低 |
| 推荐最大连接时长 | 5 min | 超过后建议客户端重连 |

关键认知：流式不会让 Agent 跑得更快——总的推理和生成时间不变。但流式将用户感知的等待时间从"全部完成才能看到"缩短为"首 token 到达即可开始阅读"，这在 Agent 动辄 10-30 秒的响应场景中对用户体验有决定性影响。
