# Memoviber

**Long-term memory for vibe coding.**

Memoviber automatically records your conversations with AI coding assistants (Cursor, Claude Code, etc.), links them to your projects, and exposes that memory through an MCP server.

> You no longer wonder *"Why did we do it like this again?"* — you just ask the AI to check Memoviber.

## Features

- **LLM Proxy** — Plugs in as a transparent proxy (LiteLLM-based)
- **Local Storage** — SQLite database per project, fully offline
- **Fast Search** — FTS5 full-text search + optional semantic embeddings
- **MCP Integration** — Expose memory to Claude/Cursor via MCP tools
- **File Tracking** — Know which conversations touched which files

## Quick Start

```bash
# Install
pip install memoviber

# Start the proxy
memoviber start

# Configure your IDE to use the proxy
# Cursor: Settings → Models → Override OpenAI Base URL → http://localhost:4000/v1
# Claude Code: export ANTHROPIC_BASE_URL=http://localhost:4000/anthropic

# Add MCP server to Claude Code
memoviber install-mcp
```

## MCP Tools

Once configured, your AI can use these tools:

| Tool | Description |
|------|-------------|
| `search_memory` | Search past conversations by keyword |
| `memory_for_file` | Get conversations that mentioned a file |
| `list_decisions` | List architectural decisions and rationale |
| `why_did_we` | Answer "why did we do X?" questions |
| `get_conversation` | Retrieve a full conversation by ID |

## How It Works

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────┐
│   IDE (Cursor,  │────▶│  Memoviber Proxy │────▶│   LLM API   │
│   Claude Code)  │     │   (LiteLLM)      │     │  (Anthropic │
└─────────────────┘     └────────┬─────────┘     │   /OpenAI)  │
                                 │               └─────────────┘
                                 ▼
                        ┌────────────────┐
                        │  SQLite DB     │
                        │  (per project) │
                        └────────┬───────┘
                                 │
                                 ▼
                        ┌────────────────┐
                        │  MCP Server    │◀──── AI queries memory
                        │  (FastMCP)     │
                        └────────────────┘
```

1. All LLM calls pass through Memoviber's proxy
2. Prompts and responses are stored in a per-project SQLite database
3. The MCP server exposes search tools to your AI assistant
4. Your AI can now query past context: decisions, discussions, code changes

## IDE Configuration

### Cursor

1. Open Settings → Models
2. Enable "Override OpenAI Base URL"
3. Enter: `http://localhost:4000/v1`
4. Your existing API key continues to work

### Claude Code

```bash
# Set environment variable
export ANTHROPIC_BASE_URL=http://localhost:4000/anthropic

# Or add to your shell profile (~/.bashrc, ~/.zshrc)
echo 'export ANTHROPIC_BASE_URL=http://localhost:4000/anthropic' >> ~/.zshrc
```

### MCP Server Registration

Add to `~/.claude/claude_code_config.json`:

```json
{
  "mcpServers": {
    "memoviber": {
      "command": "memoviber",
      "args": ["mcp"]
    }
  }
}
```

Or run:

```bash
memoviber install-mcp
```

## Configuration

```yaml
# ~/.config/memoviber/config.yaml

proxy:
  host: 127.0.0.1
  port: 4000
  upstream:
    anthropic: https://api.anthropic.com
    openai: https://api.openai.com

storage:
  path: ~/.memoviber/projects

search:
  use_embeddings: false          # Enable for semantic search
  embedding_model: all-MiniLM-L6-v2

conversation:
  session_timeout_minutes: 30    # Gap before starting new conversation
```

## CLI Commands

```bash
# Start the proxy server
memoviber start

# Start in background
memoviber start --daemon

# Check status
memoviber status

# Search memory from CLI
memoviber search "authentication flow"

# List recent conversations
memoviber list --limit 10

# Show memory for a specific file
memoviber file src/auth/login.py

# Export project memory
memoviber export --format json > memory.json

# Install MCP server config
memoviber install-mcp

# Start MCP server directly (used by Claude Code)
memoviber mcp
```

## Example Usage

Once Memoviber is running and MCP is configured, you can ask your AI:

```
"Check memoviber - why did we decide to use Redis for caching?"

"What did we discuss about the authentication flow? Check our memory."

"Search memoviber for conversations about the database schema."

"What files did we work on yesterday related to the API?"
```

## Data Storage

Memory is stored per-project in SQLite databases:

```
~/.memoviber/
├── config.yaml
└── projects/
    ├── a1b2c3d4/          # Hash of /home/user/myproject
    │   └── memory.db
    └── e5f6g7h8/          # Hash of /home/user/another-project
        └── memory.db
```

Each database contains:
- **conversations** — Grouped exchanges with timestamps
- **messages** — Individual prompts and responses
- **file_mentions** — Which files were discussed or modified
- **FTS5 index** — For fast full-text search

## Privacy

- **100% Local** — All data stays on your machine
- **No telemetry** — No data is sent anywhere
- **No cloud sync** — Everything is stored in local SQLite files
- **You own your data** — Export, backup, or delete anytime

## Requirements

- Python 3.10+
- SQLite with FTS5 (included in standard Python)

## Tech Stack

| Component | Technology |
|-----------|------------|
| Proxy | [LiteLLM](https://github.com/BerriAI/litellm) |
| MCP Server | [FastMCP](https://github.com/jlowin/fastmcp) |
| Storage | SQLite + FTS5 |
| Embeddings | [sentence-transformers](https://www.sbert.net/) (optional) |
| CLI | [Click](https://click.palletsprojects.com/) |

## Development

```bash
# Clone the repo
git clone https://github.com/yourname/memoviber.git
cd memoviber

# Install in dev mode
pip install -e ".[dev]"

# Run tests
pytest

# Run proxy in dev mode
memoviber start --reload
```

## Roadmap

- [ ] LiteLLM proxy with request/response capture
- [ ] SQLite storage with FTS5 search
- [ ] FastMCP server with basic tools
- [ ] Semantic search with local embeddings
- [ ] Automatic conversation summarization
- [ ] Web UI for browsing memory
- [ ] VS Code extension
- [ ] Memory sharing between team members (opt-in)

## License

MIT

---

**Memoviber** — Because your AI should remember what you've already built together.
