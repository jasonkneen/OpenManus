# CLAUDE.md - OpenManus AI Assistant Guide

This document provides comprehensive guidance for AI assistants working with the OpenManus codebase.

## Project Overview

OpenManus is an open-source AI agent framework that enables building general-purpose AI agents capable of solving various tasks using multiple tools. It provides a flexible architecture for creating agents that can perform programming, web browsing, file processing, information retrieval, and more.

**Key Features:**
- Multi-agent execution with planning flows
- Tool-based architecture with extensible tools
- MCP (Model Context Protocol) integration for remote tools
- Support for multiple LLM providers (OpenAI, Azure, AWS Bedrock, Anthropic, Ollama)
- Browser automation using Playwright
- Sandbox execution environment (Docker-based)

## Codebase Structure

```
OpenManus/
├── app/                        # Main application code
│   ├── agent/                  # Agent implementations
│   │   ├── base.py            # BaseAgent - abstract base class
│   │   ├── react.py           # ReActAgent - think/act pattern
│   │   ├── toolcall.py        # ToolCallAgent - tool execution
│   │   ├── manus.py           # Manus - main general-purpose agent
│   │   ├── browser.py         # Browser agent with context helper
│   │   ├── mcp.py             # MCP-enabled agent
│   │   ├── data_analysis.py   # Data analysis specialized agent
│   │   ├── sandbox_agent.py   # Sandbox execution agent
│   │   └── swe.py             # Software engineering agent
│   │
│   ├── tool/                   # Tool implementations
│   │   ├── base.py            # BaseTool, ToolResult classes
│   │   ├── tool_collection.py # ToolCollection for managing tools
│   │   ├── python_execute.py  # Python code execution
│   │   ├── bash.py            # Shell command execution
│   │   ├── browser_use_tool.py# Browser automation
│   │   ├── str_replace_editor.py # File editing
│   │   ├── web_search.py      # Web search
│   │   ├── crawl4ai.py        # Web crawling
│   │   ├── mcp.py             # MCP client tools
│   │   ├── planning.py        # Planning tool
│   │   ├── terminate.py       # Terminate execution
│   │   ├── ask_human.py       # Human interaction
│   │   ├── search/            # Search engine implementations
│   │   ├── sandbox/           # Sandbox-specific tools
│   │   └── chart_visualization/ # Data visualization tools
│   │
│   ├── flow/                   # Execution flow management
│   │   ├── base.py            # BaseFlow abstract class
│   │   ├── planning.py        # PlanningFlow implementation
│   │   └── flow_factory.py    # Flow creation factory
│   │
│   ├── prompt/                 # System prompts for agents
│   │   ├── manus.py           # Manus agent prompts
│   │   ├── toolcall.py        # ToolCall agent prompts
│   │   └── ...                # Other agent prompts
│   │
│   ├── sandbox/                # Sandbox execution environment
│   │   ├── client.py          # Sandbox client
│   │   └── core/              # Core sandbox implementation
│   │
│   ├── mcp/                    # MCP server implementation
│   │   └── server.py          # MCP server
│   │
│   ├── config.py              # Configuration management
│   ├── llm.py                 # LLM client wrapper
│   ├── schema.py              # Pydantic models (Message, Memory, etc.)
│   ├── logger.py              # Logging configuration
│   ├── bedrock.py             # AWS Bedrock client
│   └── exceptions.py          # Custom exceptions
│
├── config/                     # Configuration files
│   ├── config.example.toml    # Example configuration
│   ├── config.example-*.toml  # Provider-specific examples
│   └── mcp.example.json       # MCP server configuration
│
├── tests/                      # Test files
│   └── sandbox/               # Sandbox tests
│
├── examples/                   # Usage examples
├── protocol/                   # Protocol implementations (A2A)
├── workspace/                  # Agent workspace directory
│
├── main.py                    # Main entry point
├── run_flow.py                # Multi-agent flow runner
├── run_mcp.py                 # MCP-enabled agent runner
├── run_mcp_server.py          # MCP server runner
└── sandbox_main.py            # Sandbox agent runner
```

## Architecture Overview

### Agent Hierarchy

```
BaseAgent (abstract)
    └── ReActAgent (abstract, think/act pattern)
            └── ToolCallAgent (tool execution)
                    ├── Manus (general-purpose)
                    ├── MCPAgent (MCP-enabled)
                    ├── DataAnalysis (data analysis)
                    └── SandboxAgent (sandboxed execution)
```

### Core Components

1. **BaseAgent** (`app/agent/base.py`):
   - Manages agent state (IDLE, RUNNING, FINISHED, ERROR)
   - Memory management with messages
   - Step-based execution loop with `max_steps` limit
   - Stuck state detection and handling

2. **ReActAgent** (`app/agent/react.py`):
   - Implements think-act pattern
   - Abstract `think()` and `act()` methods
   - `step()` combines think and act

3. **ToolCallAgent** (`app/agent/toolcall.py`):
   - Handles LLM tool calling
   - Manages tool execution and results
   - Supports tool choice modes: AUTO, REQUIRED, NONE

4. **Manus** (`app/agent/manus.py`):
   - Main general-purpose agent
   - Integrates MCP tools dynamically
   - Browser context awareness
   - Factory method `create()` for async initialization

### Tool System

Tools extend `BaseTool` (`app/tool/base.py`):
- `name`: Tool identifier
- `description`: Tool description for LLM
- `parameters`: JSON schema for parameters
- `execute(**kwargs)`: Async execution method
- `to_param()`: Convert to OpenAI function format

**ToolResult** contains:
- `output`: Execution result
- `error`: Error message if failed
- `base64_image`: Optional image data
- `system`: System message

### Flow System

Flows orchestrate multi-agent execution:
- **BaseFlow**: Abstract flow with agent management
- **PlanningFlow**: Creates plans and executes steps with appropriate agents

### Configuration

Configuration uses TOML files (`config/config.toml`):
```toml
[llm]
model = "gpt-4o"
base_url = "https://api.openai.com/v1"
api_key = "YOUR_API_KEY"
max_tokens = 4096
temperature = 0.0

[llm.vision]
# Vision model configuration

[browser]
headless = false

[sandbox]
use_sandbox = false

[mcp]
server_reference = "app.mcp.server"
```

MCP servers configured in `config/mcp.json`.

## Development Workflow

### Running the Agent

```bash
# Basic agent
python main.py

# With prompt argument
python main.py --prompt "Your task here"

# Multi-agent flow
python run_flow.py

# MCP-enabled agent
python run_mcp.py

# MCP server
python run_mcp_server.py
```

### Adding a New Tool

1. Create a new file in `app/tool/`:
```python
from app.tool.base import BaseTool, ToolResult

class MyTool(BaseTool):
    name: str = "my_tool"
    description: str = "Description for the LLM"
    parameters: dict = {
        "type": "object",
        "properties": {
            "param1": {"type": "string", "description": "..."},
        },
        "required": ["param1"],
    }

    async def execute(self, param1: str, **kwargs) -> ToolResult:
        # Implementation
        return ToolResult(output="result")
```

2. Register in `app/tool/__init__.py`
3. Add to agent's `available_tools` in the agent class

### Adding a New Agent

1. Extend `ToolCallAgent` or `ReActAgent`:
```python
from app.agent.toolcall import ToolCallAgent
from app.tool import ToolCollection, Terminate

class MyAgent(ToolCallAgent):
    name: str = "MyAgent"
    description: str = "Agent description"

    system_prompt: str = "Your system prompt"
    next_step_prompt: str = "Next step guidance"

    available_tools: ToolCollection = Field(
        default_factory=lambda: ToolCollection(
            # Your tools here
            Terminate(),
        )
    )
```

2. Create prompts in `app/prompt/myagent.py`
3. Register in `app/agent/__init__.py`

## Coding Conventions

### Style Guidelines

- **Formatter**: Black (line length not specified, uses default 88)
- **Import sorting**: isort with black profile
- **Unused imports**: autoflake removes them automatically
- **Pre-commit hooks**: Run before commits

```bash
# Run pre-commit checks
pre-commit run --all-files
```

### Code Patterns

1. **Async/await**: All agent and tool execution is async
2. **Pydantic models**: Use for data validation and serialization
3. **Type hints**: Required for all function signatures
4. **Logging**: Use `app.logger.logger` for consistent logging

```python
from app.logger import logger

logger.info("Information message")
logger.warning("Warning message")
logger.error("Error message")
```

### Error Handling

- Use `ToolResult(error="message")` for tool failures
- Raise `TokenLimitExceeded` for token limit errors
- Log exceptions with `logger.exception()` or `logger.error()`

### Message Handling

Use the `Message` class factory methods:
```python
from app.schema import Message

Message.user_message("content")
Message.system_message("content")
Message.assistant_message("content")
Message.tool_message("content", name="tool_name", tool_call_id="id")
```

## Testing

Tests are located in `tests/` directory:

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/sandbox/test_sandbox.py

# Run with verbose output
pytest -v
```

Test dependencies:
- pytest
- pytest-asyncio (for async tests)

## Key Files Reference

| File | Purpose |
|------|---------|
| `app/agent/base.py` | Base agent with state management |
| `app/agent/manus.py` | Main general-purpose agent |
| `app/agent/toolcall.py` | Tool calling implementation |
| `app/tool/base.py` | Base tool and result classes |
| `app/config.py` | Configuration loading and management |
| `app/llm.py` | LLM client with token counting |
| `app/schema.py` | Core data models |
| `app/flow/planning.py` | Planning flow for multi-step tasks |

## Common Tasks

### Modifying Agent Behavior

1. **Change system prompt**: Edit `app/prompt/<agent>.py`
2. **Add tools**: Modify `available_tools` in agent class
3. **Adjust limits**: Change `max_steps`, `max_observe` in agent

### Working with MCP

MCP servers are configured in `config/mcp.json`:
```json
{
  "mcpServers": {
    "server_id": {
      "type": "stdio",
      "command": "command",
      "args": ["arg1", "arg2"]
    }
  }
}
```

### Debugging

1. Enable verbose logging in `app/logger.py`
2. Check token usage in LLM responses
3. Use `--prompt` argument for reproducible testing

## Dependencies

Key dependencies (see `requirements.txt`):
- `pydantic`: Data validation
- `openai`: LLM client
- `tenacity`: Retry logic
- `playwright`: Browser automation
- `browser-use`: Browser interaction
- `mcp`: Model Context Protocol
- `tiktoken`: Token counting
- `loguru`: Logging

## Environment Setup

```bash
# Create environment (conda)
conda create -n open_manus python=3.12
conda activate open_manus

# Or using uv (recommended)
uv venv --python 3.12
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Install playwright browsers (optional)
playwright install

# Copy and configure settings
cp config/config.example.toml config/config.toml
# Edit config/config.toml with your API keys
```

## Notes for AI Assistants

1. **Always use async/await** for agent and tool operations
2. **Check token limits** when making LLM calls
3. **Use ToolResult** for consistent tool return values
4. **Follow the agent hierarchy** when extending functionality
5. **Run pre-commit** before suggesting code changes
6. **Test changes** with the appropriate test files
7. **Update __init__.py** when adding new modules
8. **Keep prompts concise** to save tokens
9. **Handle errors gracefully** with proper logging
10. **Use factory methods** for creating agents (e.g., `Manus.create()`)
