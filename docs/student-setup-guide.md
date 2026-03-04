# EnergyPlus MCP — Student Setup Guide

Complete these steps **before** class begins. Allow 30–45 minutes for downloads and installation.

---

## Step 1: Install Docker Desktop

Docker runs the EnergyPlus MCP server in a container — no need to install EnergyPlus separately.

**Download:** https://www.docker.com/products/docker-desktop/

- Choose the installer for your operating system (Windows / Mac / Linux)
- Run the installer and follow the prompts
- **Windows users:** Ensure WSL 2 is enabled if prompted during installation
- After installation, launch Docker Desktop and wait for it to fully start (the whale icon in your system tray should stop animating)

**Verify:**
```bash
docker --version
# Expected: Docker version 24.x.x or newer
```

---

## Step 2: Install Git

Git is needed to download the EnergyPlus MCP repository.

**Download:** https://git-scm.com/downloads

Use default installation options.

**Verify:**
```bash
git --version
```

---

## Step 3: Install Claude Desktop

Claude Desktop is the AI assistant interface that connects to the MCP server.

**Download:** https://claude.ai/download

Install and sign in with your Anthropic account (free tier is fine for setup; Pro recommended for class).

---

## Step 4: Clone and Build the EnergyPlus MCP Server

There is no pre-built Docker image — you must clone the repository and build locally.

```bash
# Clone the repository
git clone https://github.com/LBNL-ETA/EnergyPlus-MCP.git

# Navigate to the devcontainer folder
cd EnergyPlus-MCP/.devcontainer

# Build the Docker image (this takes ~10 minutes — it downloads EnergyPlus 25.1.0)
docker build -t energyplus-mcp-dev .
```

**Verify the image exists:**
```bash
docker images | grep energyplus
# Expected: energyplus-mcp-dev   latest   ...   ~2GB
```

---

## Step 5: Configure Claude Desktop

You need to tell Claude Desktop how to start the MCP server.

### Find your config file

| OS | Location |
|----|----------|
| **Windows** | `%APPDATA%\Claude\claude_desktop_config.json` |
| **Mac** | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| **Linux** | `~/.config/Claude/claude_desktop_config.json` |

### Edit the config

Open the file in a text editor and set its contents to:

```json
{
  "mcpServers": {
    "energyplus": {
      "command": "docker",
      "args": [
        "run",
        "--rm",
        "-i",
        "-v", "/path/to/EnergyPlus-MCP:/workspace",
        "-w", "/workspace/energyplus-mcp-server",
        "energyplus-mcp-dev",
        "uv", "run", "python", "-m", "energyplus_mcp_server.server"
      ]
    }
  }
}
```

**Important:** Replace `/path/to/EnergyPlus-MCP` with the actual path where you cloned the repo.

**Path format examples:**

| OS | Example |
|----|---------|
| **Windows** | `C:/Users/YourName/EnergyPlus-MCP` |
| **Mac** | `/Users/YourName/EnergyPlus-MCP` |
| **Linux** | `/home/yourname/EnergyPlus-MCP` |

> ⚠️ **Windows users:** Use forward slashes (`/`) in the path, not backslashes (`\`).

---

## Step 6: Restart Claude Desktop

1. **Completely quit** Claude Desktop (not just close the window — exit from the system tray / menu bar)
2. Make sure Docker Desktop is running
3. Relaunch Claude Desktop

---

## Step 7: Verify Everything Works

1. Start a **new conversation** in Claude Desktop
2. Look for the MCP tools icon (hammer/wrench) in the interface
3. Ask Claude: **"What EnergyPlus tools do you have available?"**

Claude should respond with a list of approximately 35 tools including `load_idf_model`, `run_energyplus_simulation`, `list_zones`, etc.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Claude doesn't show MCP tools | Make sure Docker Desktop is running **before** starting Claude Desktop |
| "Cannot connect to Docker daemon" | Launch Docker Desktop and wait for it to fully start |
| JSON syntax error in config | Check for missing commas, correct quotes, and valid path format |
| "Image not found" | Re-run `docker build -t energyplus-mcp-dev .` from the `.devcontainer` folder |
| Path issues on Windows | Use forward slashes: `C:/Users/Name/...` not `C:\Users\Name\...` |
| Tools don't appear in existing chat | MCP tools only load in **new** conversations — start a fresh one |
| Server path errors (Linux/Docker paths) | If config.py has `/app/software/EnergyPlusV25-1-0`, the Docker container handles this — don't change it |

---

## Pre-Class Checklist

- [ ] Docker Desktop installed and running
- [ ] Git installed
- [ ] Claude Desktop installed and signed in
- [ ] EnergyPlus-MCP repo cloned and Docker image built
- [ ] `claude_desktop_config.json` configured with correct path
- [ ] Claude Desktop restarted — MCP tools visible in new conversation
