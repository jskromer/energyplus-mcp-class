# Setup Guide

## Part 1: Surrogate Model (Required)

The surrogate MCP server runs natively on macOS with Apple Silicon. No Docker needed.

### Install uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Clone and sync

```bash
git clone https://github.com/jskromer/ane-surrogate.git
cd ane-surrogate
uv sync
```

### Test locally

```bash
uv run mcp_server.py
```

You should see the server start. Press Ctrl+C to stop.

### Add to Claude Desktop

1. Open Claude Desktop
2. Go to **Settings > Developer > Edit Config**
3. Add the server (merge with any existing `mcpServers`):

```json
{
  "mcpServers": {
    "ane-surrogate": {
      "command": "uv",
      "args": [
        "--directory", "/absolute/path/to/ane-surrogate",
        "run", "mcp_server.py"
      ]
    }
  }
}
```

4. Replace `/absolute/path/to/ane-surrogate` with your actual clone path
5. Restart Claude Desktop

### Verify

Start a new conversation and ask:

> "What energy prediction tools do you have available?"

You should see 4 tools:
- `predict_energy` — single-month energy prediction
- `compare_scenarios` — compare 2–4 building configurations across 12 months
- `sweep_parameter` — vary one parameter across its range
- `get_parameter_info` — show valid parameter names and ranges

### Using with Claude Code

If you prefer Claude Code (CLI), add to your project's `.mcp.json`:

```json
{
  "mcpServers": {
    "ane-surrogate": {
      "command": "uv",
      "args": [
        "--directory", "/absolute/path/to/ane-surrogate",
        "run", "mcp_server.py"
      ]
    }
  }
}
```

---

## Part 2: Full EnergyPlus (Optional)

For running arbitrary EnergyPlus IDF models with full simulation fidelity, add the Docker-based EnergyPlus MCP server.

### Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)

### Setup

```bash
git clone https://github.com/LBNL-ETA/EnergyPlus-MCP.git
cd EnergyPlus-MCP/.devcontainer
docker build -t energyplus-mcp-dev .
```

The build takes ~10 minutes. Once complete, add to your Claude Desktop config alongside the surrogate:

```json
{
  "mcpServers": {
    "ane-surrogate": {
      "command": "uv",
      "args": [
        "--directory", "/absolute/path/to/ane-surrogate",
        "run", "mcp_server.py"
      ]
    },
    "energyplus": {
      "command": "docker",
      "args": [
        "run", "-i", "--rm",
        "-v", "/path/to/your/idf-files:/workspace/models",
        "energyplus-mcp-dev"
      ]
    }
  }
}
```

Restart Claude Desktop. You'll now have both surrogate tools (fast, instant) and full EnergyPlus tools (35 tools, full fidelity).

### When to use which

| | Surrogate | Full EnergyPlus |
|---|---|---|
| **Speed** | ~0.035 ms/prediction | Minutes per simulation |
| **Setup** | Native, no Docker | Requires Docker |
| **Scope** | DOE Small Office, monthly totals | Any IDF model, any output |
| **Use case** | Rapid exploration, parameter sweeps | Detailed analysis, custom models |

## Troubleshooting

**"Server not found" in Claude Desktop**
- Check that the path in your config is absolute (starts with `/`)
- Ensure `uv` is on your PATH — try running `which uv` in terminal

**"No tools available"**
- Restart Claude Desktop after editing the config
- Check the MCP server logs: **Settings > Developer > ane-surrogate > Logs**

**Docker build fails**
- Ensure Docker Desktop is running
- Try `docker system prune` to free space, then rebuild
