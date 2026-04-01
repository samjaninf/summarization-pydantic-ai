# Capabilities

The recommended way to add context management to a Pydantic AI agent is via
[capabilities](https://ai.pydantic.dev/capabilities/). No middleware wrappers needed.

## Why Capabilities?

| Feature | Capabilities | Processor API |
|---------|:-:|:-:|
| Native pydantic-ai | Yes | Yes |
| Tool output truncation | `ContextManagerCapability` | No |
| Auto-detect max_tokens | `ContextManagerCapability` | No |
| `compact()` outside run | `ContextManagerCapability` | No |
| Agent-triggered compaction | `ContextManagerCapability` (with `include_compact_tool=True`) | No |
| AgentSpec YAML | Yes | No | No |

## Available Capabilities

### ContextManagerCapability

Full context management — token tracking, auto-compression, tool output truncation:

```python
from pydantic_ai_summarization import ContextManagerCapability

cap = ContextManagerCapability(
    max_tokens=100_000,          # Auto-detected if None
    compress_threshold=0.9,      # Compress at 90% usage
    max_tool_output_tokens=5000, # Truncate large tool outputs
    include_compact_tool=True,   # Add compact_conversation tool
)
```

When `include_compact_tool=True`, the agent gets a `compact_conversation(focus?)` tool
that triggers compression on the next model request. The optional `focus` parameter
guides the summary to prioritize specific topics.

### SummarizationCapability

LLM-based history compression:

```python
from pydantic_ai_summarization import SummarizationCapability

cap = SummarizationCapability(
    trigger=("messages", 50),
    keep=("messages", 10),
)
```

### SlidingWindowCapability

Zero-cost message trimming:

```python
from pydantic_ai_summarization import SlidingWindowCapability

cap = SlidingWindowCapability(
    trigger=("messages", 100),
    keep=("messages", 50),
)
```

### LimitWarnerCapability

Warn the agent before limits hit:

```python
from pydantic_ai_summarization import LimitWarnerCapability

cap = LimitWarnerCapability(
    max_iterations=40,
    max_context_tokens=100_000,
    max_total_tokens=200_000,
)
```

## Composing Capabilities

```python
agent = Agent(
    "openai:gpt-4.1",
    capabilities=[
        LimitWarnerCapability(max_iterations=40),
        ContextManagerCapability(max_tokens=100_000),
    ],
)
```
