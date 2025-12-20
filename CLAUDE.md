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

## Docker Usage

Build and run with Docker:

```bash
# Build the image
docker build -t openmanus .

# Run interactively
docker run -it --rm \
  -v $(pwd)/config:/app/OpenManus/config \
  -v $(pwd)/workspace:/app/OpenManus/workspace \
  openmanus python main.py
```

The Dockerfile uses Python 3.12-slim and installs dependencies via `uv`.

## CI/CD and GitHub Workflows

The project uses several GitHub Actions workflows (`.github/workflows/`):

| Workflow | Purpose |
|----------|---------|
| `pre-commit.yaml` | Runs pre-commit checks on PRs |
| `build-package.yaml` | Builds the Python package |
| `environment-corrupt-check.yaml` | Validates environment setup |
| `pr-autodiff.yaml` | Auto-generates PR diffs |
| `stale.yaml` | Manages stale issues/PRs |
| `top-issues.yaml` | Tracks top issues |

## Pull Request Guidelines

When creating PRs, follow the template (`.github/PULL_REQUEST_TEMPLATE.md`):

1. **Features**: Describe features or bug fixes
2. **Feature Docs**: Link to RFCs or tutorials for significant updates
3. **Influence**: Explain the impact for reviewer focus
4. **Result**: Include screenshots or test logs
5. **Other**: Additional notes

## Public API Exports

### Agents (`app/agent/__init__.py`)
```python
from app.agent import (
    BaseAgent,
    BrowserAgent,
    MCPAgent,
    ReActAgent,
    SWEAgent,
    ToolCallAgent,
)
```

### Tools (`app/tool/__init__.py`)
```python
from app.tool import (
    BaseTool,
    Bash,
    BrowserUseTool,
    CreateChatCompletion,
    Crawl4aiTool,
    PlanningTool,
    StrReplaceEditor,
    Terminate,
    ToolCollection,
    WebSearch,
)
```

## MCP (Model Context Protocol) Details

### Running MCP Agent

```bash
# With stdio connection (default)
python run_mcp.py

# With SSE connection
python run_mcp.py --connection sse --server-url http://localhost:8000/sse

# Interactive mode
python run_mcp.py --interactive

# Single prompt
python run_mcp.py --prompt "Your task"
```

### MCP Server Configuration

Configure in `config/mcp.json`:

```json
{
  "mcpServers": {
    "server1": {
      "type": "sse",
      "url": "http://localhost:8000/sse"
    },
    "server2": {
      "type": "stdio",
      "command": "python",
      "args": ["-m", "my_mcp_server"]
    }
  }
}
```

### Starting MCP Server

```bash
python run_mcp_server.py
```

## LLM Provider Configuration

The project supports multiple LLM providers. Example configurations in `config/`:

| File | Provider |
|------|----------|
| `config.example.toml` | Default (OpenAI-compatible) |
| `config.example-model-anthropic.toml` | Anthropic Claude |
| `config.example-model-azure.toml` | Azure OpenAI |
| `config.example-model-google.toml` | Google AI |
| `config.example-model-ollama.toml` | Ollama (local) |
| `config.example-model-ppio.toml` | PPIO |
| `config.example-model-jiekouai.toml` | JiekouAI |
| `config.example-daytona.toml` | Daytona sandbox |

### AWS Bedrock Configuration

```toml
[llm]
api_type = "aws"
model = "us.anthropic.claude-3-7-sonnet-20250219-v1:0"
base_url = "bedrock-runtime.us-west-2.amazonaws.com"
api_key = "placeholder"  # Required but not used
```

## Agent State Machine

```
IDLE ──► RUNNING ──► FINISHED
           │
           ▼
         ERROR
```

- `IDLE`: Initial state, ready to accept requests
- `RUNNING`: Actively processing steps
- `FINISHED`: Task completed successfully
- `ERROR`: An error occurred during execution

## Memory Management

The `Memory` class (`app/schema.py`) manages conversation history:

```python
memory = Memory(max_messages=100)
memory.add_message(Message.user_message("Hello"))
memory.add_messages([msg1, msg2])
recent = memory.get_recent_messages(5)
memory.clear()
```

## Token Counting

The LLM class includes token counting:

```python
llm = LLM()
tokens = llm.count_tokens("text")
msg_tokens = llm.count_message_tokens(messages)
llm.update_token_count(input_tokens, completion_tokens)
```

Token limits can be configured via `max_input_tokens` in config.

## Sandbox Execution

Enable sandboxed execution for safer code running:

```toml
[sandbox]
use_sandbox = true
image = "python:3.12-slim"
work_dir = "/workspace"
memory_limit = "512m"
cpu_limit = 1.0
timeout = 300
network_enabled = false
```

## Search Engine Configuration

Configure web search behavior:

```toml
[search]
engine = "Google"  # Primary engine
fallback_engines = ["DuckDuckGo", "Baidu", "Bing"]
retry_delay = 60
max_retries = 3
lang = "en"
country = "us"
```

## Notes for AI Assistants

### Critical Guidelines

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

### Code Quality Checklist

- [ ] Type hints on all functions
- [ ] Async functions where appropriate
- [ ] Pydantic models for data structures
- [ ] Error handling with ToolResult
- [ ] Logging with `app.logger.logger`
- [ ] Tests for new functionality
- [ ] Pre-commit hooks pass
- [ ] Documentation updated

### Common Pitfalls

1. **Forgetting `await`**: All agent/tool methods are async
2. **Direct instantiation of Manus**: Use `await Manus.create()` instead
3. **Missing cleanup**: Call `agent.cleanup()` when done
4. **Ignoring token limits**: Monitor `TokenLimitExceeded` exceptions
5. **Hardcoded paths**: Use `config.workspace_root` and `config.root_path`

### File Modification Best Practices

1. **Read before editing**: Always understand existing code first
2. **Minimal changes**: Only modify what's necessary
3. **Preserve style**: Match existing code formatting
4. **Update exports**: Add to `__init__.py` when creating new modules
5. **Test thoroughly**: Ensure changes don't break existing functionality
