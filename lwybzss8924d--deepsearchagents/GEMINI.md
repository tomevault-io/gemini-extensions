## deepsearchagents

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## DeepSearchAgent Project

DeepSearchAgent is an intelligent agent system that combines the ReAct (Reasoning + Acting) framework and the CodeAct concept (executable code agents) to enable deep web search and reasoning capabilities. Built on Hugging Face's `smolagents` framework, it provides multi-step reasoning for complex queries through various interfaces.

**Version 0.3.3.dev** introduces a complete web frontend with real-time streaming, metadata-driven component routing, and a simplified WebSocket API architecture.

## Development Commands

### Common Development Tasks
```bash
# Install dependencies (with development tools)
uv pip install -e ".[dev,test,cli]"

# Run tests
make test

# Run specific test file
uv run -- pytest tests/test_specific.py

# Run tests with coverage
uv run -- pytest --cov=src tests

# Start development servers
make run        # FastAPI server (default port 8000)
# Web API v2 is accessible via the main FastAPI server
make cli        # Interactive CLI mode

# CLI with specific agent modes
make cli-react  # ReAct agent (non-interactive)
make cli-codact # CodeAct agent (non-interactive)

# MCP server
python -m src.agents.servers.run_fastmcp

# Frontend development
cd frontend && npm install   # Install dependencies
cd frontend && npm run dev   # Start development server (port 3000)
cd frontend && npm run build # Build for production
cd frontend && npm run lint  # Run linting
```

### Configuration Setup
```bash
cp config.template.toml config.toml  # Non-sensitive settings
cp .env.example .env                 # API keys
```

## High-Level Architecture

### Agent System Design
The project implements a dual-agent architecture:
- **CodeAct Agent**: Executes Python code to perform actions, suitable for computational tasks
- **ReAct Agent**: Uses structured tool calling with JSON responses, better for web search tasks
- **Runtime Manager**: Handles execution environment and tool availability for both agent types
- **Model Routing**: Supports multiple LLM providers through LiteLLM with configurable model selection
- **Web API v2**: Simplified WebSocket API with direct Gradio message pass-through (~500 lines, down from ~5000)

### Core Components Interaction
```
User Input → Agent Selection → Planning/Reasoning → Tool Execution → Response Streaming
                    ↓                    ↓                ↓
              config.toml          Prompt Templates    Toolbox Manager
                    
Web API v2 → WebSocket → Session Manager → Gradio Passthrough → stream_to_gradio → Agent
                           │
                           ▼
                    Frontend (Next.js)
                           │
                    ┌──────┴──────┐
                    │  WebSocket  │
                    │    Hook     │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  Component  │
                    │   Router    │
                    └──────┬──────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────▼────┐      ┌────▼────┐      ┌────▼────┐
    │Planning │      │ Action  │      │ Final   │
    │  Card   │      │ Thought │      │ Answer  │
    └─────────┘      └─────────┘      └─────────┘
```

### Tool Ecosystem
Tools are unified through a common interface supporting both sync/async operations:
- **Search Tools**: Multi-engine support (Google via Serper, X.com via xAI)
- **Content Processing**: URL reading, text chunking, embedding, reranking
- **Computation**: WolframAlpha integration for mathematical queries
- **Final Answer**: Structured response generation with source attribution

### Configuration Hierarchy
1. Command-line arguments (highest priority)
2. Environment variables
3. config.toml settings
4. Default values (lowest priority)

### Streaming Architecture
- Streaming is now available but disabled by default for stability
- When enabled, provides real-time visibility into agent reasoning process
- Supports both FastAPI SSE and Gradio streaming interfaces
- CLI now supports streaming output with the new StreamingConsoleFormatter

#### Enabling Streaming
To enable streaming output in the CLI:

1. **In config.toml**:
```toml
[agents.common]
cli_streaming_enabled = true  # Global toggle for CLI streaming

[agents.react]
enable_streaming = true  # Enable for React agent

[agents.codact]
enable_streaming = true  # Enable for CodeAct agent
```

2. **Via command line**:
```bash
# Enable streaming for a single query
python -m src.cli --agent-type react --enable-streaming --query "your query"
```

3. **Environment variables** (highest priority):
```bash
export REACT_ENABLE_STREAMING=true
export CODACT_ENABLE_STREAMING=true
export CLI_STREAMING_ENABLED=true
```

Note: Streaming support depends on the agent and model capabilities. Not all models support streaming output.

### Web API v2 Design Principles

The v2 API follows a simplified architecture:
1. **Direct Pass-through**: Messages from smolagents' stream_to_gradio are passed with minimal transformation
2. **Field Renaming Only**: Gradio ChatMessage fields renamed to DSAgentRunMessage format
3. **No Complex Parsing**: Avoids fragile regex-based message content parsing
4. **Leverage Proven Code**: Uses smolagents' battle-tested streaming infrastructure
5. **Session-based**: WebSocket connections maintain conversation state via session IDs

## Project Structure

```tree
src/
├── main.py                # Main application entry point
├── cli.py                 # Command-line interface
├── agents/               # Agent implementations
│   ├── __init__.py        # Package initialization
│   ├── base_agent.py      # Base agent interface with model routing
│   ├── codact_agent.py    # CodeAct agent implementation
│   ├── react_agent.py     # ReAct agent implementation
│   ├── runtime.py         # Agent runtime manager
│   ├── prompt_templates/  # Modular prompt template system
│   │   ├── __init__.py
│   │   ├── codact_prompts.py
│   │   └── react_prompts.py
│   ├── servers/           # Server implementations 
│   │   ├── __init__.py
│   │   ├── run_fastmcp.py # FastMCP MCP server implementation
│   │   └── run_fastmcp.py # FastMCP MCP server implementation
├── api/                   # FastAPI service components
│   ├── v1/                # API version 1 endpoints
│   └── v2/                # API version 2 endpoints
│       ├── __init__.py
│       ├── models.py      # Simple Pydantic models
│       ├── web_ui.py      # Agent event processing
│       ├── ds_agent_message_processor.py  # Message processor wrapper
│       ├── endpoints.py   # WebSocket and REST endpoints
│       ├── session.py     # Session management
│       ├── main.py        # Standalone API server
│       └── examples/      # Example scripts
├── core/                  # Core system components
│   ├── chunk/             # Text chunking components
│   ├── config/            # Configuration handling with Pydantic
│   ├── ranking/           # Content ranking algorithms
│   ├── scraping/          # Web content scraping (JinaAI, XCom)
│   └── search_engines/    # Search engine integrations
└── tools/                 # Tool implementations
    ├── __init__.py
    ├── chunk.py           # Text chunking tool
    ├── embed.py           # Text embedding tool
    ├── final_answer.py    # Final answer generation tool
    ├── readurl.py         # URL content reading tool
    ├── rerank.py          # Content reranking tool
    ├── search.py          # Web search tool
    ├── toolbox.py         # Tool management utilities
    └── wolfram.py         # Wolfram Alpha computational tool

tests/
└── api_v2/                # Web API v2 test suite
    ├── __init__.py
    ├── conftest.py        # Test configuration and fixtures
    ├── test_websocket_streaming.py  # WebSocket streaming tests
    ├── test_agent_steps.py          # Agent step progression tests
    ├── test_final_answer.py         # Final answer delivery tests
    └── utils.py           # Test utilities

frontend/                  # Next.js web frontend
├── app/                   # Next.js app directory
│   ├── page.tsx          # Main page component
│   ├── layout.tsx        # Root layout
│   └── globals.css       # Global styles
├── components/           # React components
│   ├── agent-chat.tsx    # Main chat interface
│   ├── action-thought-card.tsx  # Agent thinking display  
│   ├── planning-card.tsx # Planning step display
│   ├── final-answer-display.tsx # Structured answers
│   ├── tool-call-badge.tsx      # Tool execution badges
│   └── ui/               # Shadcn UI components
├── hooks/                # Custom React hooks
│   ├── use-websocket.tsx # WebSocket connection management
│   └── use-session.tsx   # Session state management
├── typings/              # TypeScript definitions
│   ├── dsagent.ts       # DSAgentRunMessage types
│   └── agent.ts         # Legacy agent types
└── utils/                # Utility functions
    └── extractors.ts     # Content extraction helpers
```

## Development Guidelines

### Code Style
- Strict PEP8 compliance with 79-character line limit
- 4-space indentation, no tabs
- English for all code, comments, and docstrings
- Chinese for user-facing chat responses only
- Import order: stdlib → third-party → local

### Testing Approach
- Test framework: pytest with asyncio support
- Test files pattern: `test_*.py` in `tests/` directory
- Both sync and async test support
- Mock external API calls in tests

### Key Development Patterns
- **Async-first design**: Tools support both sync/async interfaces
- **Type hints**: Required for all function signatures
- **Error handling**: Comprehensive try-except with specific error types
- **Logging**: Use configured loggers, not print statements
- **Configuration**: All settings through config system, no hardcoded values
- **Token Counting**: Uses smolagents v1.20.0 TokenUsage API for accurate token tracking

## Frontend Architecture

### Message Protocol

The frontend communicates via WebSocket using the DSAgentRunMessage format:

```typescript
interface DSAgentRunMessage {
  // Core fields
  message_id: string;
  role: "user" | "assistant";
  content: string;
  
  // Metadata for routing and rendering
  metadata: {
    // Component routing
    component?: "chat" | "webide" | "terminal";
    
    // Message type identification
    message_type?: string;
    step_type?: "planning" | "action" | "final_answer";
    
    // Streaming state
    streaming?: boolean;
    is_delta?: boolean;
    stream_id?: string;
    
    // UI-specific fields
    thoughts_content?: string;  // First 60 chars of thinking
    has_structured_data?: boolean;
    answer_title?: string;
    tool_name?: string;
  };
}
```

### Component Routing

Messages are routed to UI components based on metadata:

- **Chat Component** (`component: "chat"`):
  - Planning messages (badges and content)
  - Action thoughts (truncated with special formatting)
  - Tool call badges
  - Final answers with structured data
  
- **Code Editor** (`component: "webide"`):
  - Python code execution with syntax highlighting
  
- **Terminal** (`component: "terminal"`):
  - Execution logs and command outputs

### Streaming Message Handling

The frontend handles streaming with delta accumulation:

1. **Initial Message**: Empty content with `streaming: true`
2. **Delta Updates**: Incremental content with `is_delta: true`
3. **Final Message**: Complete content with `streaming: false`

Messages are accumulated by `stream_id` in the WebSocket hook.

## Testing

### Backend Testing
```bash
# Run all tests
make test

# Run API v2 tests specifically
uv run -- pytest tests/api_v2/

# Test WebSocket streaming
uv run -- pytest tests/api_v2/test_websocket_streaming.py -v
```

### Frontend Testing
```bash
# Frontend unit tests
cd frontend && npm test

# E2E WebSocket testing
python debug-archive/scripts/test_websocket_e2e.py
python debug-archive/scripts/test_frontend_ui_fixes.py
python debug-archive/scripts/test_action_thought_e2e.py
```

### Debug Mode

Enable debug logging in the frontend:

```typescript
// In frontend/hooks/use-websocket.tsx
const DEBUG_PLANNING = true;
```

## Development Best Practices

### Frontend Development

1. **Metadata-First Approach**: Always use metadata for routing decisions, not content parsing
2. **Type Safety**: Use TypeScript interfaces from `typings/dsagent.ts`
3. **Component Isolation**: Each component handles its own rendering logic
4. **Streaming Support**: Handle both streaming and non-streaming modes gracefully
5. **WebSocket Resilience**: Implement automatic reconnection and error handling

### Message Processing

1. **Trust Backend Metadata**: The backend provides authoritative routing information
2. **Handle Unknown Types**: Always provide fallback rendering for unknown message types
3. **Debug Strategically**: Add targeted debug logs but clean up after feature completion
4. **Test Both Agents**: Ensure features work with both ReAct and CodeAct agents

## Git Workflow

### Creating New Local Development Branches

When creating new development branches, follow this workflow:

#### Branch Naming Convention
- Format: `v{version}-dev.{YYMMDD}.{task-description}`
- Examples:
  - `v0.3.3-dev.250730.debug-ui-empty-chatbox`
  - `v0.3.3-dev.250731.ui-style-optimization`
  - `v0.3.3-dev.250801.api-performance-improvement`

#### Workflow Steps

1. **Create Summary of Current Work**:
   - Document all resolved issues/features with task IDs
   - Format: `[RESOLVED] Task(N): [YYMMDD-N] Type: {description}`
   - Example:
     - `[RESOLVED] Task(1): [250730-1] Issues: Empty ChatBox UI`
     - `[RESOLVED] Task(2): [250730-2] Features: Tool call badges`

2. **Check Git Status & Stage Changes**:
   ```bash
   git status
   git diff --cached  # Review staged changes
   ```
   - Ensure all major code modifications are staged
   - Include relevant documentation and test files
   - Use `git add -p` for selective staging if needed

3. **Create Comprehensive Commit**:
   - Follow Conventional Commits specification
   - Include detailed commit message with:
     - Summary of fixed issues
     - Technical changes made
     - Testing performed
   - Add co-author attribution:
     ```
     🤖 Generated with [Claude Code](https://claude.ai/code)
     
     Co-Authored-By: Claude <noreply@anthropic.com>
     ```

4. **Create New Branch**:
   ```bash
   git checkout -b v{version}-dev.{YYMMDD}.{task-type}
   ```
   - Task types: features, debug, test, experiment, integration, pr, release
   - Branch from current work to maintain continuity

#### Example Workflow
```bash
# 1. Review and stage changes
git status
git add frontend/components/*.tsx
git add src/api/v2/web_ui.py
git add debug-archive/

# 2. Create comprehensive commit
git commit -m "fix(frontend): resolve empty chatbox and tool badges

[RESOLVED] Task(1): [250730-1] Empty ChatBox UI
[RESOLVED] Task(2): [250730-2] Tool call badges verification

- Filter separator messages causing empty boxes
- Verify tool extraction from CodeAct Python code
- Add comprehensive test infrastructure

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>"

# 3. Create new branch for next task
git checkout -b v0.3.3-dev.250731.ui-style-optimization
```

---
> Source: [lwyBZss8924d/DeepSearchAgents](https://github.com/lwyBZss8924d/DeepSearchAgents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-17 -->
