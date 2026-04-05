# CoPaw Trajectory Collector

Agent Trajectory Collector for generating CoPaw-Flash training datasets. Uses a dual-AI architecture where both user and assistant roles are played by AI models.

## Overview

This tool collects agent trajectories in the CoPaw-Flash format for fine-tuning language models on tool-use and data analysis tasks. It leverages the claude-code-clean framework for stable agent execution.

### Key Features

- **Dual-AI Architecture**: User Agent simulates non-technical users, Analyst Agent performs data analysis
- **CoPaw-Flash Format**: Output compatible with ms-swift training pipeline
- **OpenAI-Compatible API**: Works with OpenRouter, DeepSeek, Ollama, and other providers
- **Real Tool Execution**: Bash, Read, Write, Glob, Grep, Edit tools with actual execution
- **Python Data Analysis**: Pre-configured uv venv with pandas, numpy, matplotlib, seaborn

## Quick Start

### Prerequisites

- [Bun](https://bun.sh) (v1.3+)
- [uv](https://github.com/astral-sh/uv) (for Python environment)
- Linux/macOS/WSL2

### Installation

```bash
# Install dependencies
bun install

# Setup Python analysis environment
uv venv .venv-analysis --python 3.12
uv pip install pandas numpy matplotlib seaborn scipy --python .venv-analysis/bin/python
```

### Run Trajectory Collection

```bash
CLAUDE_CODE_USE_OPENAI=1 \
OPENAI_API_KEY=your-api-key \
OPENAI_BASE_URL=https://openrouter.ai/api/v1 \
OPENAI_MODEL=qwen/qwen3.6-plus:free \
bun run collect \
  --dataset-dir ./test_data \
  --dataset-title "Sales Data Analysis" \
  --output-dir ./trajectory_output \
  --max-turns 3
```

### Command Line Options

| Option | Description | Default |
|--------|-------------|---------|
| `--dataset-dir` | Directory containing data files (CSV, JSON, etc.) | `./data` |
| `--dataset-title` | Title shown to the AI analyst | `Dataset` |
| `--dataset-desc` | Description of the dataset | `A dataset for analysis` |
| `--output-dir` | Directory for output trajectories | `./trajectory_output` |
| `--max-turns` | Maximum conversation turns | `3` |
| `--workspace-dir` | Directory for generated scripts/charts | `<dataset-dir>/workspace` |

## Output Format

### JSONL (Training Format)

```json
{"messages":[
  {"role":"user","content":"Help me analyze this sales data..."},
  {"role":"system","content":"You are a professional Data Analyst..."},
  {"role":"assistant","content":"<tool_call>\n{\"name\": \"Glob\", \"arguments\": {\"pattern\": \"*.csv\"}}\n</tool_call>"},
  {"role":"user","content":"<tool_response>\nsales_data.csv\n</tool_response>"},
  {"role":"assistant","content":"I found the data file. Let me read it..."}
]}
```

### Tool Call Format (CoPaw-Flash Compatible)

```xml
<tool_call>
{"name": "Bash", "arguments": {"command": "python analysis.py"}}
</tool_call>
```

### Tool Response Format

```xml
<tool_response>
Analysis complete. Total revenue: $162,270.15
</tool_response>
```

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                  Trajectory Collector                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐         ┌──────────────┐                  │
│  │  User Agent  │ ──────> │Analyst Agent │                  │
│  │ (Simulates   │         │ (Data        │                  │
│  │  non-tech    │ <────── │  Analysis)   │                  │
│  │  user)       │         │              │                  │
│  └──────────────┘         └──────┬───────┘                  │
│                                  │                           │
│                           ┌──────▼───────┐                  │
│                           │ Tool Executor │                  │
│                           │ Bash/Read/    │                  │
│                           │ Write/Glob/   │                  │
│                           │ Grep/Edit     │                  │
│                           └──────┬───────┘                  │
│                                  │                           │
│                           ┌──────▼───────┐                  │
│                           │   Python     │                  │
│                           │ Environment  │                  │
│                           │ (uv venv)    │                  │
│                           └──────────────┘                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │ Output Files    │
                    │ - .jsonl        │
                    │ - .json (full)  │
                    └─────────────────┘
```

## Performance

Based on test runs with `qwen/qwen3.6-plus:free`:

| Metric | Value |
|--------|-------|
| Time per trajectory | ~4 minutes |
| Input tokens | ~50,000 |
| Output tokens | ~5,000 |
| Messages generated | ~20 |
| Tool calls | ~10-15 |

## Available Tools

| Tool | Description |
|------|-------------|
| **Bash** | Execute shell commands, run Python scripts |
| **Read** | Read file contents with line numbers |
| **Write** | Create/overwrite files |
| **Glob** | Find files by pattern |
| **Grep** | Search for patterns in files |
| **Edit** | Edit existing files (find & replace) |

## Project Structure

```
copaw-trajectory-collector/
├── src/
│   ├── entrypoints/
│   │   └── trajectoryCollect.tsx   # Main entry point
│   └── trajectory/
│       ├── collector.ts            # Trajectory recording
│       └── dualAgent.ts            # Dual-AI coordination
├── test_data/                      # Sample datasets
│   └── sales_data.csv
├── trajectory_output/              # Generated trajectories
├── .venv-analysis/                 # Python environment
├── docs/                           # Documentation
└── README.md
```

## Documentation

See the `docs/` directory for detailed documentation:

- [Data Collection Guide](docs/data-collection-guide.md) - How to collect trajectories
- [Output Format Specification](docs/output-format.md) - Dataset format details
- [API Configuration](docs/api-configuration.md) - Setting up different LLM providers

## Based On

This project is based on [claude-code-clean](https://github.com/IIIIQIIII/claude-code-clean), a privacy-focused fork of Anthropic's Claude Code with all telemetry removed.

## License

MIT License - See LICENSE file for details.

---

**Version:** 1.0.0
**Last Updated:** 2026-04-05
