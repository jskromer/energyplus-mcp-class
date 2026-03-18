# EnergyPlus MCP Training

Class materials for learning building energy simulation with EnergyPlus through the Model Context Protocol (MCP).

## What This Is

This course teaches you to use Claude as an AI-powered interface for building energy analysis. You interact conversationally — predicting energy use, comparing design scenarios, and sweeping parameters — using natural language instead of manually editing IDF files.

The primary tool is a **surrogate model** that runs on Apple's Neural Engine, delivering instant predictions (~0.035 ms each) trained on EnergyPlus simulation data. For students who want full-fidelity simulation, an optional Docker-based EnergyPlus MCP server is also available.

## Repository Structure

```
├── README.md              # This file
├── docs/
│   └── setup-guide.md     # Setup instructions (surrogate + optional EnergyPlus)
└── exercises/
    └── (coming soon)
```

## Quick Start

### Prerequisites

- macOS with Apple Silicon (M1/M2/M3/M4)
- [Claude Desktop](https://claude.ai/download) or [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [uv](https://docs.astral.sh/uv/getting-started/installation/) (Python package manager)
- [Git](https://git-scm.com/downloads)

### 1. Clone the surrogate server

```bash
git clone https://github.com/jskromer/ane-surrogate.git
cd ane-surrogate
uv sync
```

### 2. Configure Claude Desktop

Open **Settings > Developer > Edit Config** and add:

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

Replace `/absolute/path/to/ane-surrogate` with the actual path where you cloned the repo.

### 3. Restart Claude Desktop

Restart the app. You should see the MCP tools icon (hammer) in the chat input area.

### Verify

Ask Claude:

> "What energy prediction tools do you have available?"

You should see 4 tools: `predict_energy`, `compare_scenarios`, `sweep_parameter`, and `get_parameter_info`.

## What the Surrogate Can Do

The surrogate models a **DOE Reference Small Office** building and predicts monthly electricity and natural gas consumption. You can vary:

- **Climate** — across all U.S. climate zones
- **Wall/roof insulation** — R-value ranges
- **Window properties** — U-factor, SHGC, window-to-wall ratio
- **Lighting and equipment** — power density (W/ft²)
- **HVAC setpoints** — heating and cooling temperatures
- **Infiltration** — air changes per hour

Try asking Claude things like:
- *"Compare energy use in Miami vs Duluth for this office building"*
- *"What happens to electricity if I double the lighting power density?"*
- *"Sweep cooling setpoint from 72°F to 80°F and show me the impact"*

## Optional: Full EnergyPlus via Docker

For full-fidelity simulation with arbitrary IDF models, you can add the [LBNL-ETA EnergyPlus-MCP](https://github.com/LBNL-ETA/EnergyPlus-MCP) server. This requires Docker and takes longer to set up. See [docs/setup-guide.md](docs/setup-guide.md) for instructions.

## Resources

- [ane-surrogate](https://github.com/jskromer/ane-surrogate) — Neural Engine surrogate MCP server
- [LBNL-ETA EnergyPlus-MCP](https://github.com/LBNL-ETA/EnergyPlus-MCP) — Full EnergyPlus MCP server (Docker)
- [EnergyPlus Documentation](https://energyplus.net/documentation)
- [MCP Protocol Spec](https://modelcontextprotocol.io)

## License

MIT
