# EnergyPlus MCP Training

Class materials for learning building energy simulation with EnergyPlus through the Model Context Protocol (MCP).

## What This Is

This course teaches you how to use Claude as an AI-powered interface for EnergyPlus building energy simulation. Instead of manually editing IDF files and running command-line simulations, you interact with EnergyPlus conversationally through Claude — loading models, modifying parameters, running simulations, and analyzing results using natural language.

The integration is powered by [MCP (Model Context Protocol)](https://modelcontextprotocol.io), which connects Claude to a containerized EnergyPlus 25.1.0 environment with 35 tools for simulation, analysis, and visualization.

## Repository Structure

```
├── README.md                          # This file
├── docs/
│   ├── student-setup-guide.md         # Pre-class setup instructions for students
│   ├── instructor-setup-guide.md      # AWS WorkSpaces cloud environment setup
│   └── troubleshooting.md             # Common issues and solutions
└── exercises/
    └── README.md                      # Exercise descriptions (coming soon)
```

## Quick Start (Local Setup)

### Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Git](https://git-scm.com/downloads)
- [Claude Desktop](https://claude.ai/download)

### Setup

```bash
# 1. Clone the EnergyPlus MCP server
git clone https://github.com/LBNL-ETA/EnergyPlus-MCP.git

# 2. Build the Docker image (~10 min)
cd EnergyPlus-MCP/.devcontainer
docker build -t energyplus-mcp-dev .

# 3. Configure Claude Desktop (see docs/student-setup-guide.md for details)

# 4. Restart Claude Desktop with Docker running
```

### Verify

In a new Claude Desktop conversation, ask:
> "What EnergyPlus tools do you have available?"

You should see 35 tools including `load_idf_model`, `run_energyplus_simulation`, `list_zones`, etc.

## Cloud Setup (For Instructors)

If you're running a class and want zero-friction student environments, see [docs/instructor-setup-guide.md](docs/instructor-setup-guide.md) for AWS WorkSpaces deployment.

## Resources

- [LBNL-ETA EnergyPlus-MCP](https://github.com/LBNL-ETA/EnergyPlus-MCP) — The MCP server
- [DOE Prototype Building Models](https://www.energycodes.gov/prototype-building-models) — Sample IDF files (v22.1.0, requires version update)
- [EnergyPlus Documentation](https://energyplus.net/documentation)
- [MCP Protocol Spec](https://modelcontextprotocol.io)

## License

MIT
