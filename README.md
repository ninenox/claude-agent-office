# Pixel Agent Office 🏢

> [🇹🇭 ภาษาไทย](README.th.md)

Pixel-art office dashboard showing an AI agent team working in real-time.
Each agent walks to different zones (desk, whiteboard, break room) based on their current task status — with waypoint pathfinding to navigate around room walls.

**Two modes:**
- **Manual** — assign tasks to each agent individually
- **Auto** — describe a goal once; the Boss AI analyzes and distributes work automatically

<img src="examples/example.gif" width="100%" alt="Pixel Agent Office">

---

## Quick Start

```bash
git clone https://github.com/ninenox/pixel-agent-office.git
cd pixel-agent-office
bash install.sh
source .venv/bin/activate
```

```bash
export ANTHROPIC_API_KEY=sk-ant-...   # if using Anthropic models
python main.py
```

Open browser: http://localhost:19000

---

## UI Layout

```
┌──────────────┬───────────────────────────────┬──────────────┐
│ TASK DISPATCH│                               │ ACTIVITY LOG │
│  (left panel)│      Pixel Office Canvas      │              │
│              │                               │ QUICK ACTIONS│
│  ⚙ MANUAL    │                               │              │
│  ✦ AUTO      ├───────────────────────────────┤              │
│              │ 📄 OUTPUT  [Agent1][Agent2].. │              │
└──────────────┴───────────────────────────────┴──────────────┘
```

- **Left panel** — Task Dispatch: assign tasks per agent (Manual) or describe a goal (Auto)
- **Center** — Pixel office canvas with animated agents, expands to fill available space
- **Output panel** — shows each agent's full response after a task; click a tab to switch
- **Right sidebar** — Activity Log and Quick Actions (toggle with ◀ Panel button)

---

## Usage

### ⚡ MANUAL Mode

Select the **⚙ MANUAL** tab in the left Task Dispatch panel.

- Type a task in each agent's input box
- Click **▶ RUN** to send a task to one agent, or press `Ctrl+Enter`
- Click **🚀 DISPATCH ALL** to send tasks to all agents simultaneously
- Click **■** next to an agent's name card to stop that agent, or **■ STOP ALL** to stop everyone

### ✦ AUTO Mode (Brainstorm)

Select the **✦ AUTO** tab in the Task Dispatch panel.

- Describe your goal in the text box
- Click **✦ BRAINSTORM** — the Boss AI analyzes and assigns tasks automatically
- All agents walk to the whiteboard first, then Boss distributes work
- The Boss's plan and per-agent assignments appear below the button

### 📄 Output Panel

The output panel below the canvas displays each agent's full response after a task completes.

- Click an agent tab to view their output
- A dot next to each tab glows while that agent is running
- Click the header to collapse/expand the panel

### CLI

**Run agents without the UI:**
```bash
python main.py --agents-only --tasks tasks.json
```

Example `tasks.json`:
```json
{
  "research-agent": "Analyze AI Agent trends for 2026",
  "sa-agent":       "Design a REST API for task management",
  "officer-agent":  "Summarize recent developments in LLMs"
}
```

**Single agent:**
```bash
cd agents
python agent_runner.py sa-agent "Design a database schema for a blog"
python agent_runner.py officer-agent "List 3 prompt engineering techniques" --stream
```

---

## Agents

Agents are defined in `config/team.json`. The UI loads agent configuration dynamically from `/team` at startup — **no code changes needed**. Add, remove, or rename agents by editing `team.json` only, then refresh the browser.

| Field | Description |
|-------|-------------|
| `name` | Display name shown in the UI |
| `role` | Short role description |
| `model` | Model ID to use |
| `provider` | `anthropic` \| `openai` \| `ollama` |
| `base_url` | Custom API endpoint (for ollama or compatible APIs) |
| `color` | Agent color (hex) |
| `system_prompt` | Full system prompt defining capabilities and behavior |
| `tools` | List of tools to enable, e.g. `["all"]` or `["web_search", "read_file"]` |

### Supported Providers

| Provider | API Key | Notes |
|----------|---------|-------|
| `anthropic` | `ANTHROPIC_API_KEY` | Default |
| `openai` | `OPENAI_API_KEY` | GPT-4o, o1, etc. |
| `ollama` | — | Local models via Ollama |
| custom | `{PROVIDER}_API_KEY` | Any OpenAI-compatible endpoint |

### Default team (team.json)

```json
{
  "research-agent": {
    "name": "qwen2.5 Researcher",
    "model": "qwen2.5:7b",
    "provider": "ollama",
    "base_url": "http://localhost:11434/v1",
    "color": "#f97316"
  },
  "sa-agent": {
    "name": "qwen2.5 SA",
    "model": "qwen2.5:7b",
    "provider": "ollama",
    "base_url": "http://localhost:11434/v1",
    "color": "#8b5cf6"
  },
  "officer-agent": {
    "name": "qwen2.5 Officer",
    "model": "qwen2.5:7b",
    "provider": "ollama",
    "color": "#06b6d4"
  }
}
```

### Example: adding an Anthropic agent

```json
"claude-opus": {
  "name": "Claude Opus",
  "role": "Senior Researcher",
  "model": "claude-opus-4-6",
  "provider": "anthropic",
  "color": "#f97316",
  "system_prompt": "You are a senior researcher..."
}
```

### Example: adding an OpenAI agent

```json
"gpt-agent": {
  "name": "GPT Agent",
  "role": "OpenAI Assistant",
  "model": "gpt-4o",
  "provider": "openai",
  "color": "#10b981",
  "system_prompt": "You are an OpenAI assistant named gpt-agent..."
}
```

### Example: enabling tools for an agent

```json
"sa-agent": {
  "tools": ["all"]
}
```

Or restrict to specific tools:

```json
"sa-agent": {
  "tools": ["run_python", "read_file", "write_file"]
}
```

### Configuring the Boss (AUTO mode)

The Boss AI analyzes goals and distributes tasks to agents. Configure it with a `"boss"` key in `team.json`:

```json
"boss": {
  "provider": "ollama",
  "model": "qwen2.5:7b",
  "base_url": "http://localhost:11434/v1",
  "system_prompt": "คุณคือ Team Lead ของทีม Agent Office..."
}
```

The Boss character is always visible seated at the head of the meeting room table.

---

## Tools

Agents can use tools to interact with the outside world. Tools live in `agents/tools/` and are **auto-discovered** on startup — add a new file and restart, no other changes needed.

### Built-in Tools

| Tool | Description | Requires |
|------|-------------|----------|
| `read_file` | Read a file from `workspace/` | — |
| `write_file` | Write a file to `outputs/` (write or append) | — |
| `run_python` | Execute Python code, returns stdout (15s timeout) | — |
| `http_request` | HTTP GET / POST / PUT / DELETE | — |
| `web_search` | Real-time web search via Brave Search API | `BRAVE_API_KEY` |

`workspace/` — place input files here for agents to read.
`outputs/` — agents write results here; also saved automatically after each task.

### Adding a New Tool

Create `agents/tools/my_tool.py`:

```python
from .base import BaseTool

class MyTool(BaseTool):
    name = "my_tool"
    description = "What this tool does"
    input_schema = {
        "type": "object",
        "properties": {
            "input": {"type": "string", "description": "..."},
        },
        "required": ["input"],
    }

    def run(self, input: str) -> str:
        return f"result: {input}"
```

Restart the server — `my_tool` is now available to all agents.

### Running an Agent with Tools (CLI)

```bash
cd agents
python agent_tools.py sa-agent "Research the latest LLM benchmarks and write a report" --tools all
python agent_tools.py sa-agent "Write and run a script to sort a CSV file" --tools run_python write_file
```

---

## CLI Options

```
python main.py
  (no flags)                      Start server; assign tasks via the web UI
  --agents-only --tasks <file>    Run agents from a JSON file (no UI)
```

---

## API

| Method | Path | Description |
|--------|------|-------------|
| GET | `/team` | Get team config from `team.json` (used by UI on load) |
| GET | `/status` | Get all agent statuses (includes `output` field when available) |
| POST | `/status` | Update a single agent's status |
| POST | `/run` | Send tasks and run agents in the background |
| POST | `/brainstorm` | Boss analyzes a goal and assigns work (AUTO mode) |
| POST | `/stop` | Stop agent(s) and set status to idle |
| GET | `/health` | Health check |

**POST `/run`:**
```json
{ "tasks": { "sa-agent": "Design...", "research-agent": "Analyze..." } }
```

**POST `/brainstorm`:**
```json
{ "task": "Analyze and design a complete e-commerce system" }
```

**POST `/stop`:**
```json
{ "agent_id": "sa-agent" }   // omit to stop all agents
```

**GET `/status` response:**
```json
{
  "agents": {
    "sa-agent": {
      "status": "idle",
      "detail": "Done ✓ [ollama]",
      "updated_at": "14:32:01",
      "output": "Here is my analysis..."
    }
  }
}
```

---

## Status → Zone

| Status | Room | Zone |
|--------|------|------|
| `writing` | Research | Desk |
| `coding` | Dev | Desk 1 |
| `researching` | Dev | Desk 2 |
| `thinking` / `planning` | Meeting | Whiteboard |
| `idle` / `error` | Break Room | — |

---

## License

MIT
