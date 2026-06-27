# IS4490-MCP-detailed

A deeper dive into Model Context Protocol (MCP) servers and why they are critical to AI initiatives.

## Overview

MCP servers give AI clients a standard way to connect with tools, data, prompts, and application workflows. Instead of hard-coding every integration into a single chatbot or agent, MCP lets teams expose capabilities through a shared protocol that clients can discover and call safely.

This repository is intended for a more advanced discussion of MCP server design. The goal is to move beyond a "hello world" tool call and think about the engineering practices required when MCP becomes part of a real AI system.

## Learning Goals

After working through this material, you should be able to:

- Explain how MCP servers expose tools, resources, and prompts to AI clients.
- Distinguish between a simple demo server and a production-ready MCP service.
- Identify advanced server capabilities such as structured schemas, authentication, progress reporting, logging, and error handling.
- Describe why observability is essential for AI systems that call external tools.
- Understand how LangSmith can be used to trace, debug, evaluate, and monitor MCP-driven workflows.

## Why MCP Servers Matter

Modern AI applications rarely operate in isolation. They need access to systems of record, documents, APIs, databases, business processes, and other software services. MCP provides a common interface for those connections.

An MCP server can:

- Publish tools that an AI client can call, such as creating a ticket, querying inventory, or running a calculation.
- Expose resources that give the client structured access to documents, database records, files, or application state.
- Provide reusable prompts that standardize how clients perform common tasks.
- Send structured errors and status updates so the client can recover gracefully.
- Create a governance boundary between an AI assistant and sensitive enterprise systems.

In a business process context, MCP is useful because it separates "what the AI can do" from "how the business system actually works." That separation makes integrations easier to audit, test, reuse, and evolve.

## Advanced MCP Server Capabilities

A basic MCP server might expose one tool and return a static response. A more capable server handles the realities of business software.

### 1. Capability Negotiation and Dynamic Discovery

MCP clients and servers negotiate capabilities when they connect. This lets a client discover what the server can provide instead of assuming that every server supports the same features.

Advanced servers should make discovery useful by exposing accurate descriptions for:

- Available tools.
- Readable resources.
- Reusable prompts.
- Supported transports.
- Progress and notification behavior.
- Server name, version, and protocol compatibility.

Dynamic discovery matters when available tools depend on user permissions, environment, feature flags, or downstream system health.

### 2. Structured Tool Interfaces

Tools should have clear names, descriptions, input schemas, and predictable outputs. The schema is not just documentation; it helps the AI client decide when to call the tool and how to format valid arguments.

Good tool design includes:

- Specific tool names that reflect business actions.
- Required and optional parameters with meaningful descriptions.
- Strong validation before touching downstream systems.
- Consistent response shapes for success, partial success, and failure.
- Error messages that are safe to show to a user or useful to an AI client.

### 3. Resources and Context

Resources allow a server to expose context without turning every data access pattern into a tool. For example, a server might expose policy documents, customer records, product specifications, or workflow state as MCP resources.

Resources are especially useful when the AI client needs to read or inspect information before deciding what action to take.

### 4. Prompt Templates

Prompt templates let a server publish reusable instructions for common workflows. This is useful when an organization wants consistent behavior across multiple AI clients.

Examples include:

- A standard prompt for summarizing a customer support case.
- A prompt for comparing a purchase order against policy.
- A prompt for generating a process improvement recommendation.

### 5. Human-in-the-Loop Workflows

Some server operations need more than a direct tool call. A server may need confirmation, missing information, or a human approval step before taking action.

Examples include:

- Asking the user to confirm before sending an email, issuing a refund, or updating a record.
- Requesting missing fields before a tool can run.
- Returning a pending state while an approval workflow completes.
- Logging who approved an action and when it happened.

These patterns are especially important in business process automation because the AI client may suggest an action, but the MCP server is responsible for enforcing the process rules.

### 6. Authentication and Authorization

Production MCP servers should not assume that every client or user can access every tool. Authorization should be enforced inside the server, not delegated entirely to the AI client.

Important questions include:

- Which users can call which tools?
- Which records can a user or agent read?
- Which actions require approval before execution?
- How are credentials stored, rotated, and scoped?
- How are denied requests logged for audit purposes?

### 7. Transports and Remote Deployment

Local demo servers often run over standard input and output. Production servers are more likely to run remotely, use HTTP-based transport, and serve multiple clients.

Remote deployment raises additional concerns:

- TLS and secure network configuration.
- OAuth, bearer tokens, API keys, or other authentication mechanisms.
- Request size limits.
- Rate limits and quotas.
- Tenant isolation.
- Backward-compatible protocol and schema changes.
- Operational dashboards and alerting.

### 8. Progress, Logging, and Errors

Long-running work needs progress reporting. Unreliable networks and external APIs need retry logic. Failed tool calls need structured errors. These details matter because the AI client needs enough information to decide whether to retry, ask the user for clarification, or stop.

Useful server behavior includes:

- Returning clear validation errors for bad input.
- Recording downstream API failures.
- Exposing progress for multi-step operations.
- Including correlation IDs so logs can be connected across systems.
- Separating internal diagnostic detail from user-facing error text.

### 9. Observability

Observability is the difference between "the agent did something weird" and "we can see exactly which tool was called, with which inputs, which system responded, how long it took, and where the failure happened."

For MCP servers, observability should cover:

- Incoming MCP requests.
- Tool selection and execution.
- Resource reads.
- Prompt usage.
- Calls to downstream APIs, databases, and services.
- Latency, retries, failures, and timeouts.
- User, session, environment, and deployment metadata.
- Human or automated feedback about output quality.

## Observability with LangSmith

LangSmith is an observability, evaluation, and monitoring platform for LLM applications. It is useful for MCP servers because MCP sits at the boundary between AI reasoning and real business action.

With LangSmith, an MCP server can record traces for tool calls and related work. A trace gives you a timeline of what happened during a request. Within a trace, individual runs or spans can represent steps such as input validation, tool execution, a database lookup, an API call, or an LLM call.

### How LangSmith Maps to MCP

| LangSmith concept | MCP server equivalent |
| --- | --- |
| Project | A server, course demo, environment, or deployment |
| Trace | One user request, tool invocation, or agent workflow |
| Run/span | A step inside the workflow, such as validation, retrieval, or an API call |
| Tags | Labels such as `mcp`, `tool`, `demo`, `production`, or `fall-2026` |
| Metadata | Structured context such as tool name, protocol version, user role, tenant, or correlation ID |
| Feedback | Human review, automated score, success flag, or quality rating |

### What to Trace

At minimum, trace the server-side boundary where MCP requests enter the system and where tool calls touch external systems.

Recommended trace points:

- MCP request received.
- Tool name selected.
- Tool input validated.
- Authorization checked.
- Resource or prompt loaded.
- External API, database, or service called.
- Result transformed for the AI client.
- Error, retry, timeout, or policy denial recorded.

Avoid logging secrets, raw credentials, private student data, protected customer data, or other sensitive values. Prefer redacted payloads, stable identifiers, and metadata that helps debugging without exposing confidential information.

### Example LangSmith Configuration

For a Python-based MCP server, the common configuration pattern is environment variables:

```bash
export LANGSMITH_TRACING=true
export LANGSMITH_API_KEY="your-langsmith-api-key"
export LANGSMITH_PROJECT="is4490-mcp-detailed"
```

The exact code depends on the MCP server framework and language, but the pattern is the same: wrap meaningful server operations in traced functions and attach useful metadata.

```python
from langsmith import traceable


@traceable(
    name="mcp.tool.lookup_order",
    run_type="tool",
    tags=["mcp", "orders"],
    metadata={"server": "is4490-mcp-detailed", "tool": "lookup_order"},
)
async def lookup_order(order_id: str) -> dict:
    """Example tool function that would be called by an MCP server."""
    order = await fetch_order_from_system(order_id)
    return {
        "order_id": order["id"],
        "status": order["status"],
        "last_updated": order["last_updated"],
    }
```

For a more complete server, trace both the MCP handler and the downstream tool function. That makes it possible to see whether a problem came from protocol handling, validation, authorization, business logic, or an external dependency.

### Useful Metadata for MCP Traces

Useful metadata makes traces searchable and explainable later.

Consider capturing:

- `mcp_server_name`
- `mcp_server_version`
- `mcp_protocol_version`
- `transport`
- `tool_name`
- `resource_uri`
- `prompt_name`
- `environment`
- `correlation_id`
- `user_role`
- `tenant_id` or course section
- `approval_required`
- `approval_status`

Use identifiers that are safe to store. If a field might contain sensitive data, redact it before sending it to any observability platform.

### Questions LangSmith Helps Answer

LangSmith is useful because it helps teams answer operational and governance questions, such as:

- Which MCP tools are called most often?
- Which tool calls fail most frequently?
- Are failures caused by the AI client, the MCP server, or a downstream system?
- Which tool calls are slow?
- Did the server enforce authorization correctly?
- What input led to a bad or surprising result?
- Did a recent prompt, model, schema, or server change make behavior better or worse?

### Teaching Example

In a classroom or workshop, a useful exercise is to compare two versions of the same MCP server:

1. A server with no tracing.
2. A server with LangSmith tracing, tags, metadata, and feedback.

Students can then run the same tool calls against both versions and compare the debugging experience. The lesson is usually immediate: once an AI system can take action, observability becomes a requirement rather than a nice-to-have.

## Production Readiness Checklist

Before treating an MCP server as production-ready, consider whether it has:

- Clear tool, resource, and prompt descriptions.
- Input validation and structured error responses.
- Authentication and authorization.
- Secrets management.
- Rate limiting and timeout handling.
- Audit logging for sensitive operations.
- Observability through traces, logs, and metrics.
- LangSmith projects, tags, metadata, and feedback where LLM behavior needs review.
- Tests for successful calls, invalid inputs, denied actions, and downstream failures.
- Documentation for operators and client developers.

## References

- [Model Context Protocol documentation](https://modelcontextprotocol.io/docs)
- [LangSmith documentation](https://docs.langchain.com/langsmith/)
- [LangSmith observability concepts](https://docs.langchain.com/langsmith/observability-concepts)
- [LangSmith tracing guide](https://docs.langchain.com/langsmith/annotate-code)
