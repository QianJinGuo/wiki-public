---
title: "Agent Security Architecture"
created: 2026-05-21
updated: 2026-08-01
type: concept
tags: [security, agent, authentication, authorization, vault, privacy, trust]
sources:
  - raw/articles/anthropic-12-mcp-production-patterns
summary: Agent 安全架构，涵盖认证授权、凭证管理、权限边界、沙箱隔离、数据隐私等核心安全考量，以及生产级 Agent 的安全设计模式。
---

# Agent Security Architecture

> 生产级 Agent 的安全边界设计是 Agent 工程中最容易被忽视但最关键的领域之一。

## 一、安全威胁模型

### 1.1 Agent 的独特威胁

| 威胁类型 | 描述 | 风险 |
|---------|------|------|
| **提示注入** | 恶意指令注入用户输入 | 数据泄露、非法操作 |
| **工具滥用** | Agent 被诱导执行危险操作 | 系统破坏 |
| **权限提升** | 突破预期的权限边界 | 未授权访问 |
| **数据泄露** | 敏感信息进入输出 | 隐私侵犯 |
| **上下文污染** | 通过上下文操纵行为 | 错误决策 |

### 1.2 攻击向量

```
用户输入 → 系统提示 → Agent 决策 → 工具调用 → 外部系统
              ↑
        提示注入攻击
              ↑
        上下文污染
```

## 二、认证与授权

### 2.1 认证模式

| 模式 | 描述 | 适用场景 |
|------|------|---------|
| **API Key** | 静态密钥认证 | 简单场景 |
| **OAuth** | 第三方授权 | 需要用户授权的系统 |
| **CIMD** | 可发现的客户端元数据 | MCP 协议 |
| **Vault** | 托管凭证 | 生产环境 |

### 2.2 MCP 认证模式

```python
# CIMD - Client ID Metadata Documents
class MCPAuth:
    def discover_auth(self, server_metadata_url):
        # 1. 获取服务器元数据
        metadata = fetch(server_metadata_url)
        
        # 2. 发现支持的认证方式
        auth_methods = metadata.get('auth_methods')
        
        # 3. 按标准流程启动 OAuth
        return start_oauth_flow(auth_methods)
```

### 2.3 凭证托管（Vault）

**问题**：token 放在工具参数、环境变量里 → 生产系统危险

```python
# 凭证不进入工具调用路径
class VaultCredentialManager:
    def create_session(self, vault_id, scopes):
        # 凭证生命周期由平台管理
        session = platform.create_session(vault_id, scopes)
        return session  # 只有 session ID 流通
    
    def inject_credential(self, session, mcp_connection):
        # 平台在运行时注入凭证
        platform.inject(session, mcp_connection)
```

## 三、权限边界

### 3.1 最小权限原则

```python
class PermissionBoundary:
    def __init__(self):
        self.tools = {}  # 工具注册
        self.policies = {}  # 权限策略
    
    def check(self, agent_id, tool, params):
        # 1. 检查 agent 是否有权限使用工具
        if not self.has_permission(agent_id, tool):
            return DENIED
        
        # 2. 检查参数是否在允许范围内
        if not self.validate_params(tool, params):
            return DENIED
        
        # 3. 检查操作类型是否允许
        if not self.is_operation_allowed(tool, params):
            return DENIED
        
        return ALLOWED
```

### 3.2 操作风险分级

| 级别 | 示例 | 要求 |
|------|------|------|
| **低** | 读数据、搜索 | 自动执行 |
| **中** | 写数据、创建资源 | 日志记录 |
| **高** | 删除、支付、部署 | 显式确认 |
| **极高** | 系统管理、凭证操作 | 多重审批 |

### 3.3 高风险操作处理

```python
# MCP Form Mode - 用户确认
class HighRiskHandler:
    def handle(self, operation):
        if operation.risk_level == HIGH:
            # 暂停执行，请求用户确认
            form = create_confirmation_form(operation)
            user_response = yield to_user(form)
            if user_response.approved:
                return execute(operation)
            else:
                return cancelled()
```

## 四、沙箱隔离

### 4.1 工具执行沙箱

```python
class SandboxedTool:
    def __init__(self):
        self.limits = {
            'memory_mb': 512,
            'cpu_seconds': 10,
            'network': False,
            'filesystem': '/tmp/sandbox'
        }
    
    def execute(self, code):
        # 1. 资源限制
        # 2. 网络隔离
        # 3. 文件系统限制
        # 4. 超时控制
        # 5. 结果过滤
        return sandbox.run(code, **self_limits)
```

### 4.2 MCP Server 隔离

```python
# 远程 MCP Server 隔离
class RemoteMCPServer:
    def __init__(self, server_url):
        self.url = server_url
        self.cert = verify_certificate()
    
    def call_tool(self, tool_name, params):
        # 凭证不经过 Client，直接由 Server 持有
        # Client 只接收 session token
        return self._secure_request(tool_name, params)
```

### 4.3 数据隔离

```python
class DataIsolation:
    def filter_output(self, raw_output, sensitivity_level):
        if sensitivity_level == HIGH:
            return redact_sensitive(raw_output)
        elif sensitivity_level == MEDIUM:
            return mask_pii(raw_output)
        else:
            return raw_output
```

## 五、提示注入防护

### 5.1 注入类型

| 类型 | 示例 |
|------|------|
| **直接注入** | "Ignore previous instructions, ..." |
| **上下文污染** | 在用户输入中植入指令 |
| **角色扮演** | 扮演系统管理员 |
| **越狱** | 通过角色扮演绕过限制 |

### 5.2 防护策略

```python
class PromptInjectionGuard:
    def __init__(self):
        self.patterns = [
            "ignore previous",
            "disregard instructions",
            "new system prompt"
        ]
    
    def check(self, user_input):
        for pattern in self.patterns:
            if pattern.lower() in user_input.lower():
                return InjectionAlert(pattern)
        return None
    
    def sanitize(self, user_input):
        # 移除检测到的注入模式
        return remove_injection_patterns(user_input)
```

### 5.3 上下文隔离

```python
class ContextIsolator:
    def build_context(self, user_input, system_prompt):
        # 1. 清理用户输入
        clean_input = self.guard.check(user_input)
        
        # 2. 强化系统提示
        reinforced_system = self.add_security_instructions(system_prompt)
        
        # 3. 分离不同来源的内容
        return {
            'system': reinforced_system,
            'user': clean_input,
            'memory': self.get_task_memory(),
            'tools': self.get_tool_definitions()
        }
```

## 六、安全最佳实践

1. **凭证托管**：敏感凭证不要放在参数或环境变量中
2. **最小权限**：Agent 只应有完成任务的最小权限
3. **操作审计**：所有操作都要有日志记录
4. **高风险确认**：危险操作必须用户确认
5. **输入清理**：用户输入必须经过安全检查
6. **输出过滤**：敏感数据必须过滤或脱敏
7. **沙箱执行**：危险工具在隔离环境中执行

## 相关实体

- [[entities/ai-agents-security-survey-attack-defense]]
- [[entities/agent-security-three-step-sequence-harness-governance-identity-crewai]]
- [[entities/where-openclaw-security-is-heading-openclaw-blog]]
- [[entities/cloudflare-glasswing-mythos-security]]
- [[entities/agent-harness-architecture-deep-dive-aksahy]]

## 七、关联实体

- [[entities/anthropic-12-mcp-production-patterns]] — MCP 安全模式
- [[concepts/agent-security-full-lifecycle-system]] — Agent 安全全生命周期
- [[concepts/model-context-protocol-mcp]] — MCP 协议与安全
- [[concepts/tool-use-patterns-ai-agents]] — 工具使用安全
- [[queries/ai-safety-threat-vectors-and-mitigation-strategies|AI Safety 威胁向量与防护策略]] — 威胁向量全景图与三层九项防护框架

## 关联实体

**上游依赖**:
- [[entities/ai-agents-security-survey-attack-defense]] — 提供基础理论/方法
- [[entities/agent-security-three-step-sequence-harness-governance-identity-crewai]] — 提供基础理论/方法
- [[entities/where-openclaw-security-is-heading-openclaw-blog]] — 提供基础理论/方法

**下游应用**:
- [[entities/cloudflare-glasswing-mythos-security]] — 具体应用场景
- [[entities/agent-harness-architecture-deep-dive-aksahy]] — 具体应用场景
- [[entities/anthropic-12-mcp-production-patterns]] — 具体应用场景

**平行协作**:
- [[entities/discord-e2e-encryption]] — 替代/补充方案
- [[entities/from-ssh-to-rest-a-security-driven-modernization-of-slacks-e]] — 替代/补充方案
- [[entities/incendium-fuzzing-ms-rpc]] — 替代/补充方案


→ [[raw/articles/anthropic-12-mcp-production-patterns|原文存档]]

## 新增关联实体
- [[entities/aigatewayproductionindex]]
- [[entities/discord-e2e-encryption]]
- [[entities/from-ssh-to-rest-a-security-driven-modernization-of-slacks-e]]
- [[entities/incendium-fuzzing-ms-rpc]]
- [[entities/openhuman-private-ai-runtime-from-openclaw]]

## 所属 MOC

- [[moc/cybersecurity-privacy|Cybersecurity Privacy]]
- [[moc/layer-5-production-security|Layer 5 Production Security]]
