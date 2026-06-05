---
title: "Agent 鉴权与授权"
description: "Agent 系统的身份认证与授权机制全景：OAuth 2.1 演进历史、SSO/OIDC 委托模式、CIBA 后台认证、Token Exchange 换票、DPoP 持有证明，以及各方案的横向技术选型对比"
tags: ["authentication", "authorization", "OAuth", "SSO", "CIBA", "Token", "security"]
date: 2025-07-01
author: "Agent Knowledge Book"
---

# Agent 鉴权与授权

## 为什么 Agent 系统的鉴权问题特殊

传统 Web 应用的认证模型假设**人类用户直接操作**——用户在浏览器中输入密码、点击授权按钮、扫码确认。但 Agent 系统打破了这个假设：Agent 代表用户自主运行，可能在用户不在场时执行操作、访问多个服务、甚至将任务委派给子 Agent。

这带来了几个独特挑战：

- **用户不在场**：Agent 在后台运行时，如何获取授权？无法弹出登录页面
- **权限委托链**：Agent A 委派任务给 Agent B，B 需要的权限如何安全传递？
- **最小权限**：Agent 可能只需要用户权限的一个子集，如何精确限定？
- **长时间运行**：Agent 任务可能持续数小时甚至数天，Token 过期怎么办？
- **高风险操作审批**：Agent 遇到敏感操作时，如何异步获取用户确认？

本章系统介绍 Agent 鉴权领域的核心技术方案，并提供横向对比指导技术选型。

## OAuth 2.1：从 2.0 到 2.1 的演进

### 为什么叫 2.1 而不是 3.0？

OAuth 2.1 的命名反映了它的本质定位：**不是全新协议，而是对 OAuth 2.0 十余年实践经验的整合与精简**。

IETF OAuth 工作组明确声明不会开发 OAuth 3.0（见 oauth.net/3/）。原因是：OAuth 2.1 并未引入全新的协议架构，而是将散布在多个 RFC/BCP 中的安全最佳实践"硬编码"为强制要求。被视为"下一代授权协议"的实际上是 **GNAP（Grant Negotiation and Authorization Protocol，RFC 9635，2024 年 10 月发布）**——它不与 OAuth 2.0 兼容，是真正意义上的架构重新设计。

### OAuth 2.0（2012）到 OAuth 2.1 之间的关键演进

```
2012  RFC 6749  OAuth 2.0 核心框架
      RFC 6750  Bearer Token 使用规范
          │
2015  RFC 7636  PKCE（修补 Authorization Code 拦截攻击）
          │
2017  RFC 8252  Native Apps 的 OAuth 最佳实践
          │
2018  RFC 8414  Authorization Server Metadata（自动发现）
          │
2019  RFC 8628  Device Authorization Grant（智能设备/CLI 场景）
          │
2020  RFC 8693  Token Exchange（换票机制）
      RFC 8705  mTLS Client Authentication + Token 证书绑定
          │
2023  RFC 9449  DPoP（Demonstrating Proof of Possession）
      RFC 9396  RAR（Rich Authorization Requests）
          │
2025.01 RFC 9700  OAuth 2.0 Security BCP（安全最佳实践集大成）
          │
2025    draft-ietf-oauth-v2-1-15  OAuth 2.1（Active Draft，技术要求已稳定）
```

### OAuth 2.1 整合了什么

OAuth 2.1（draft-ietf-oauth-v2-1-15）将以下规范整合为单一文档：

- RFC 6749（OAuth 2.0 核心）— 基础框架
- RFC 7636（PKCE）— 从"可选扩展"变为"强制要求"
- RFC 8252（Native Apps）— 浏览器和原生应用最佳实践
- RFC 9700（Security BCP）— 十年安全经验总结

### OAuth 2.1 废弃了什么

| 被废弃的内容 | OAuth 2.0 中的状态 | 废弃原因 |
|---|---|---|
| **Implicit Grant**（response_type=token） | 推荐给 SPA（单页应用） | Token 暴露在 URL fragment 中，可通过浏览器历史/Referer 头/日志泄露 |
| **Resource Owner Password Credentials** | 允许使用 | 违反委托授权核心原则——应用直接收集用户的用户名和密码 |
| **Token 放在 URL query string** | 允许 | 通过 Web 服务器日志、Referer header、浏览器缓存等途径泄露 |

新增强制要求：

- **PKCE 对所有客户端强制**：包括 Confidential Client（之前只要求 Public Client）
- **Redirect URI 精确匹配**：禁止通配符和路径前缀匹配
- **Refresh Token 约束**：必须是 sender-constrained（绑定到客户端）或 one-time-use（每次使用后轮换）

### OAuth 2.1 在 Agent/MCP 中的应用

MCP 规范（2025-03-26 版本）**明确选择 OAuth 2.1 作为 HTTP 传输的官方授权标准**。

核心流程：

```
MCP Client                    MCP Server              Authorization Server
    │                              │                          │
    │  POST /mcp (tools/call)      │                          │
    ├─────────────────────────────►│                          │
    │  HTTP 401 Unauthorized       │                          │
    │◄─────────────────────────────┤                          │
    │                              │                          │
    │  生成 PKCE: code_verifier + code_challenge               │
    │                              │                          │
    │  GET /authorize?response_type=code                       │
    │  &client_id=mcp-client&code_challenge=...               │
    ├──────────────────────────────────────────────────────────►│
    │                              │                          │
    │  (User 登录 + 授权)           │                          │
    │                              │                          │
    │  302 Redirect: ?code=abc123  │                          │
    │◄──────────────────────────────────────────────────────────┤
    │                              │                          │
    │  POST /token: code=abc123&code_verifier=...             │
    ├──────────────────────────────────────────────────────────►│
    │  { access_token: "...", refresh_token: "..." }           │
    │◄──────────────────────────────────────────────────────────┤
    │                              │                          │
    │  POST /mcp + Bearer Token    │                          │
    ├─────────────────────────────►│  验证 Token              │
    │  200 OK (tool result)        │                          │
    │◄─────────────────────────────┤                          │
```

MCP 还支持 **RFC 7591 动态客户端注册**——因为 Agent/Client 无法预知所有可能连接的 MCP Server，需要在首次连接时动态注册自身。

## SSO（单点登录）与 Agent 委托

### SAML 2.0 vs OIDC 对比

| 维度 | SAML 2.0 | OIDC（OpenID Connect） |
|------|----------|----------------------|
| 发布时间 | 2005（OASIS 标准） | 2014（OpenID Foundation） |
| 数据格式 | XML（SAML Assertion） | JSON（JWT） |
| 传输方式 | HTTP POST/Redirect Binding | HTTP REST + JSON |
| Token 类型 | XML 签名的 Assertion | ID Token（JWT）+ Access Token |
| 协议基础 | 独立协议 | 建立在 OAuth 2.0 之上 |
| 适用场景 | 企业级 SSO（ADFS、Okta SAML） | 移动/Web/API/微服务 |
| 动态发现 | 无标准化发现 | `.well-known/openid-configuration` |
| API 友好度 | 差（面向浏览器重定向） | 好（原生 REST/JSON） |
| Agent 适配性 | 差（需要浏览器交互） | 好（支持多种非浏览器 flow） |

**结论**：Agent 系统应优先选择 OIDC。SAML 2.0 仅在需要与遗留企业 IdP（如 ADFS）集成时考虑。

### Agent 场景下的 SSO 委托模式

Agent 代表用户行事时，核心挑战是**如何将用户的身份和权限安全传递给 Agent**。

**模式一：前置 SSO + Token 传递**

用户通过 SSO 登录 → 获得 Access Token → Token 传递给 Agent → Agent 持 Token 访问下游服务。简单但缺乏灵活性（Agent 拥有与用户完全相同的权限）。

**模式二：Token Exchange + 权限缩减**

用户 SSO 登录后，Agent 通过 Token Exchange（RFC 8693）将用户的宽权限 Token 交换为受限的 delegated token：

```http
POST /token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:token-exchange
&subject_token=eyJ...（用户的 access_token）
&subject_token_type=urn:ietf:params:oauth:token-type:access_token
&requested_token_type=urn:ietf:params:oauth:token-type:access_token
&audience=downstream-service-A
&scope=read:data
```

返回的新 Token 包含 `act`（actor）claim，记录"谁在代替谁操作"：

```json
{
  "sub": "user-alice",
  "scope": "read:data",
  "aud": "downstream-service-A",
  "act": {
    "sub": "agent-orchestrator-001"
  }
}
```

**模式三：On-Behalf-Of（OBO）流程**

Microsoft Entra（Azure AD）成熟的实现。Agent 将用户的 Access Token 作为 assertion 提交给授权服务器，换取一个新 Token：subject 是用户，actor 是 Agent，scope 可以收窄。

### Impersonation vs Delegation

| 模式 | Token 中的身份 | 下游可见性 | 审计能力 | 推荐度 |
|------|--------------|----------|---------|--------|
| **Impersonation（模拟）** | sub=用户，无 act claim | 下游无法区分是用户还是 Agent | 差 | 不推荐 |
| **Delegation（委托）** | sub=用户 + act=Agent | 下游知道"Agent X 代表用户 Y" | 好 | 推荐 |

### 多 Agent 委托链

当 Orchestrator Agent 委派任务给 Sub-Agent 时，Token 形成嵌套的 actor 链：

```json
{
  "sub": "user-alice",
  "act": {
    "sub": "agent-orchestrator",
    "act": {
      "sub": "agent-code-reviewer"
    }
  }
}
```

IETF 新兴草案 `draft-niyikiza-oauth-attenuating-agent-tokens` 专门解决多 Agent 链中 Token 逐级衰减（Attenuation）的问题——每经过一级委托，权限自动收窄。

## CIBA：后台认证（Client-Initiated Backchannel Authentication）

### 为什么 CIBA 对 Agent 特别重要

CIBA 的核心设计是**将"认证发起方"与"用户交互方"解耦**——在一个设备发起请求，在另一个设备完成用户确认。这完美匹配了 Agent 的核心场景：

- Agent 在云端/后台自主运行
- Agent 遇到高风险操作（转账、删除数据、发布代码）
- Agent 需要用户批准，但用户不在 Agent 所在的设备上
- 通过 CIBA，用户在手机上收到推送通知："Agent 想执行 [操作]，是否批准？"

### CIBA 三种 Token 交付模式

```
┌─────────────┐        ┌──────────────────┐         ┌────────────────┐
│   Agent     │        │ Authorization    │         │  User's Phone  │
│  (Client)   │        │    Server (OP)   │         │  (Auth Device) │
└──────┬──────┘        └────────┬─────────┘         └───────┬────────┘
       │                        │                           │
       │ POST /bc-authorize     │                           │
       │ {login_hint,           │                           │
       │  binding_message,      │                           │
       │  scope, acr_values}    │                           │
       ├───────────────────────►│                           │
       │                        │   Push / SMS / App        │
       │ 202 Accepted           │   "Agent wants to        │
       │ {auth_req_id,          │    transfer $5000.        │
       │  expires_in, interval} │    Code: AGENT-7X9K"      │
       │◄───────────────────────├──────────────────────────►│
       │                        │                           │
       │    [Poll / Ping / Push 三选一]                      │
       │                        │   User taps [Approve]     │
       │                        │◄──────────────────────────┤
       │                        │                           │
       │ POST /token            │                           │
       │ grant_type=urn:openid: │                           │
       │  params:grant-type:ciba│                           │
       │ &auth_req_id=...       │                           │
       ├───────────────────────►│                           │
       │                        │                           │
       │ {access_token,         │                           │
       │  id_token, ...}        │                           │
       │◄───────────────────────┤                           │
       │                        │                           │
       │ Agent continues        │                           │
       │ execution with token   │                           │
       └────────────────────────┘                           │
```

| 模式 | 机制 | Client 行为 | 适用场景 |
|------|------|-------------|---------|
| **Poll** | Client 周期性轮询 Token 端点 | 每隔 `interval` 秒 POST /token | 实现最简单，适合大多数场景 |
| **Ping** | OP 通知 Client 再取 Token | Client 暴露通知端点，收到 ping 后取 Token | 减少无效轮询，适合高并发 |
| **Push** | OP 直接推送 Token 到 Client | Client 被动接收 Token（HTTPS 端点） | 延迟最低，但 Client 须暴露端点 |

### 真实案例：Auth0 for AI Agents

Auth0 已发布 `auth0-ai-js` SDK，将 CIBA 封装为 Agent 友好的 "Async Authorization"：

```typescript
import { CIBAAuthorizer } from "@auth0/ai";

const cibaAuth = new CIBAAuthorizer({
  domain: "your-domain.auth0.com",
  clientId: "agent-client-id",
  clientSecret: "agent-secret",
});

// Agent 发现需要执行高风险操作
async function executeHighRiskAction() {
  // 发起 CIBA 请求 — 用户手机会收到推送通知
  const result = await cibaAuth.authorize({
    loginHint: "user@example.com",
    bindingMessage: "Approve transfer of $5,000 to account ending 1234",
    scope: "transfer:execute",
    acrValues: "urn:mace:incommon:iap:silver"  // 要求中等强度认证
  });
  
  if (result.approved) {
    // 使用获得的 access_token 执行操作
    await transferFunds(result.accessToken, { amount: 5000, to: "...1234" });
  } else {
    // 用户拒绝或超时
    await notifyOrchestrator("high_risk_action_denied");
  }
}
```

### CIBA + RAR（Rich Authorization Requests）

在 Agent 场景中，CIBA 常与 RAR（RFC 9396）配合，在授权请求中携带结构化的操作描述，让用户在审批时看到具体操作细节：

```json
{
  "authorization_details": [{
    "type": "payment_initiation",
    "actions": ["execute"],
    "locations": ["https://bank.example/accounts/1234"],
    "instructedAmount": { "currency": "USD", "amount": "5000.00" },
    "creditorName": "Merchant Corp",
    "initiatedBy": "agent-orchestrator-001"
  }]
}
```

用户在手机上看到的不是模糊的"是否授权"，而是明确的"Agent 要向 Merchant Corp 转账 $5000，是否批准？"

## 核心概念详解

### Token 类型

| Token 类型 | 格式 | 用途 | 生命周期 | Agent 场景 |
|---|---|---|---|---|
| **Access Token** | Opaque 或 JWT | 访问受保护资源 | 短（分钟～小时） | Agent 调用 API/Tool 的凭证 |
| **Refresh Token** | Opaque | 获取新 Access Token | 长（天～月） | Agent 长时间运行时静默续期 |
| **ID Token** | JWT（必须） | 证明用户身份（OIDC） | 一次性 | Agent 首次获取用户身份信息 |

**JWT 结构示例（Agent 的 Delegated Access Token）：**

```json
{
  "header": {
    "alg": "RS256",
    "typ": "at+jwt",
    "kid": "key-2025-06"
  },
  "payload": {
    "iss": "https://auth.example.com",
    "sub": "user-alice-123",
    "aud": "https://api.example.com",
    "exp": 1750000000,
    "iat": 1749996400,
    "scope": "read:data write:reports",
    "client_id": "agent-orchestrator",
    "act": {
      "sub": "agent-orchestrator",
      "client_id": "mcp-client-claude"
    },
    "cnf": {
      "jkt": "SHA-256 thumbprint of DPoP public key"
    }
  }
}
```

### Token Exchange（RFC 8693）— "换票"

Token Exchange 定义了标准化的 Security Token Service (STS) 协议，核心解决"我有 Token A，需要换成适用于不同场景的 Token B"。

**典型 Agent 换票场景：**

```
┌──────────────┐     ┌───────────────────┐     ┌──────────────┐
│ User Token   │     │  Authorization    │     │ Agent Token  │
│ (宽权限)     │────►│     Server        │────►│ (窄权限)     │
│ scope=full   │     │  Token Exchange   │     │ scope=read   │
│ aud=any      │     │                   │     │ aud=service-A│
└──────────────┘     └───────────────────┘     └──────────────┘
```

请求格式：

```http
POST /token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:token-exchange
&subject_token=eyJ...用户Token
&subject_token_type=urn:ietf:params:oauth:token-type:access_token
&actor_token=eyJ...Agent自身凭证（可选）
&actor_token_type=urn:ietf:params:oauth:token-type:jwt
&audience=downstream-service-A
&scope=read:data
&requested_token_type=urn:ietf:params:oauth:token-type:access_token
```

### DPoP — Proof of Possession（持有证明）

DPoP（RFC 9449）解决 Bearer Token 的核心缺陷：**Token 被窃取后可被任何人使用**。

工作原理：

1. Agent 生成非对称密钥对（如 ES256/P-256），私钥安全存储
2. 获取 Token 时，将公钥绑定到 Token（授权服务器在 Token 中写入 `cnf.jkt` = 公钥指纹）
3. 每次使用 Token 时，Agent 用私钥签名一个 DPoP Proof JWT：

```json
{
  "typ": "dpop+jwt",
  "alg": "ES256",
  "jwk": { "kty": "EC", "crv": "P-256", "x": "...", "y": "..." }
}
{
  "jti": "unique-request-id",
  "htm": "POST",
  "htu": "https://api.example.com/execute",
  "iat": 1750000000,
  "ath": "base64url(SHA-256(access_token))"
}
```

4. Resource Server 验证：DPoP Proof 的签名密钥 == Token 中绑定的公钥。**不一致则拒绝**。

这意味着即使 Token 被网络中间人截获，没有 Agent 的私钥就无法使用——Token 和 Agent 身份绑定在一起。

### mTLS 客户端证书

Mutual TLS 在 TLS 握手层面实现双向认证：

- 传统 TLS：只验证服务器证书（Server → Client 信任链）
- mTLS：客户端也必须出示证书，服务器验证客户端身份

**Agent 场景用途：**

- Agent 间 Service Mesh 通信的强身份认证
- 证书由企业 PKI 签发，绑定到具体 Agent 实例/工作负载
- 配合 OAuth：mTLS 作为 client authentication method（代替 client_secret），Token 通过 `cnf.x5t#S256` 绑定到证书
- RFC 8705 定义了完整的 OAuth 2.0 + mTLS 集成规范

**局限性**：证书生命周期管理复杂（签发、轮换、吊销），不适合动态扩缩容场景。推荐配合 SPIFFE/SPIRE 等 workload identity 框架简化证书管理。

### API Key vs OAuth Token

| 维度 | API Key | OAuth Token |
|------|---------|-------------|
| 复杂度 | 极低（一个字符串） | 中-高（需要授权流程） |
| 权限粒度 | 粗粒度（通常全有或全无） | 细粒度（scope / claims / audience） |
| 生命周期 | 长期（除非手动轮换） | 短期 + 自动续期（Refresh Token） |
| 撤销 | 需人工操作 | 即时（introspection / short-lived） |
| 用户上下文 | 无（只标识应用） | 有（可携带用户身份） |
| 审计 | 仅知道哪个 key 被使用 | 知道谁(sub)、通过谁(act)、做了什么(scope) |
| 盗用风险 | 高（长期有效 + 无绑定） | 低（短期 + DPoP/mTLS 绑定） |
| 适用场景 | 内部简单工具、低风险 API | 用户数据、跨组织、高安全要求 |

### Service-to-Service：Client Credentials Flow

Agent 以自身身份（非代表用户）访问服务资源：

```http
POST /token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&client_id=agent-service-001
&client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer
&client_assertion=eyJ...(private_key_jwt 签名)
&scope=inventory:read store:query
```

返回的 Token `sub` 是客户端自身（Agent），不包含用户身份。OAuth 2.1 推荐使用 `private_key_jwt` 或 mTLS 代替 `client_secret`（密钥不应在网络上传输）。

## 横向技术选型对比

| 维度 | OAuth 2.1 (AuthZ Code) | SSO/OIDC | CIBA | API Key | mTLS |
|------|---|---|---|---|---|
| **安全等级** | 高 | 高 | 很高 | 低-中 | 很高 |
| **用户体验** | 首次浏览器交互，后台续期 | 一次登录多服务通用 | 异步推送，无需 Agent 设备操作 | 无感 | 无感 |
| **实现复杂度** | 中 | 中-高 | 高 | 极低 | 高 |
| **Agent 典型场景** | Agent 代表用户访问 MCP Server/SaaS | 企业 Agent 统一身份 | 高风险操作用户实时批准 | 简单工具/内部 API | Agent 间 Service Mesh |
| **延迟** | 首次有交互，后续 ms 级 | 首次 SSO，后续 ms 级 | 等待用户（秒～分钟） | 极低 | 低（+1-2 RTT） |
| **Token 委托** | 支持（RFC 8693） | 支持（OBO） | 操作级授权 | 不支持 | 需配合 OAuth |
| **离线 Agent** | 需 Refresh Token | Session 过期问题 | 天然适合 | 天然适合 | 天然适合 |
| **审计能力** | 强 | 强 | 很强（操作级） | 弱 | 中 |
| **标准/RFC** | draft-ietf-oauth-v2-1 | OIDC Core 1.0 | OIDC CIBA Core 1.0 | 无统一标准 | RFC 8705 |

## 场景化推荐

**场景 A — Agent 代表用户访问外部服务（如 MCP Tool）：**
OAuth 2.1 Authorization Code + PKCE，配合 DPoP 绑定 Token + Token Exchange 权限缩减。

**场景 B — Agent 执行高风险操作需用户确认：**
CIBA + RAR（Rich Authorization Requests），通过 `binding_message` 展示操作详情。

**场景 C — Agent 间通信（微服务/子 Agent）：**
Client Credentials + mTLS 或 private_key_jwt，短生命周期 Token + 最小 scope。

**场景 D — 快速原型/内部工具集成：**
API Key（带 IP 白名单 + 速率限制），后续过渡到 OAuth Client Credentials。

**场景 E — 多 Agent 委托链（Orchestrator → Sub-Agent）：**
Token Exchange（RFC 8693）+ `act` claim 嵌套，参考 IETF draft-niyikiza-oauth-attenuating-agent-tokens 实现逐级衰减。

## 新兴 IETF 草案（2025 年 Agent 鉴权方向）

| 草案 | 内容 | 状态 |
|------|------|------|
| draft-oauth-ai-agents-on-behalf-of-user | AI Agent 获取 delegated access token 的 OAuth 扩展 | Active Draft |
| draft-liu-agent-operation-authorization | Agent 操作授权框架，引入操作请求 JWT | Active Draft |
| draft-niyikiza-oauth-attenuating-agent-tokens | 多 Agent 链中 Token 逐级权限衰减 | Active Draft |

这些草案反映了 IETF 社区对 Agent 鉴权问题的高度关注，预计将在 2025-2026 年逐步成熟并影响 MCP/A2A 等协议的认证设计。

## 参考

- [IETF] "The OAuth 2.1 Authorization Framework" — draft-ietf-oauth-v2-1-15
- [IETF] RFC 8693 — OAuth 2.0 Token Exchange
- [IETF] RFC 9449 — OAuth 2.0 DPoP (Demonstrating Proof of Possession)
- [IETF] RFC 9635 — GNAP (Grant Negotiation and Authorization Protocol)
- [IETF] RFC 9700 — OAuth 2.0 Security Best Current Practice
- [OpenID Foundation] "CIBA Core 1.0" — Client-Initiated Backchannel Authentication
- [Auth0, 2025] "auth0-ai-js: Authentication for AI Agents"
- [Anthropic, 2025] "MCP Specification — Authorization" (2025-03-26)
- 相关章节：[MCP 协议深度解析](../08-multi-agent/mcp-and-protocols.md)、[威胁模型](./threat-model.md)、[权限控制与沙箱](./permission-control.md)
