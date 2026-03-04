# EnergyPlus MCP — Instructor Setup Guide

How to deploy a zero-friction cloud training environment using AWS WorkSpaces.

---

## Why Cloud Deployment?

Running classes locally means debugging Docker, paths, and OS quirks on every student's laptop. AWS WorkSpaces gives each student an identical pre-configured Ubuntu desktop accessible from any browser — no local installs required.

---

## Architecture

```
┌──────────────────────────────────────────────────┐
│              AWS WorkSpaces (Ubuntu)              │
│  ┌────────────────────────────────────────────┐  │
│  │  Claude Code (terminal)                    │  │
│  │       ↕ MCP Protocol                       │  │
│  │  Docker Container                          │  │
│  │  ├── EnergyPlus 25.1.0                     │  │
│  │  ├── EnergyPlus MCP Server                 │  │
│  │  └── Python + Dependencies                 │  │
│  └────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────┘
         ↑ Web Client / RDP
┌─────────────────────┐
│  Student's Browser   │
└─────────────────────┘
```

---

## Cost Estimate

| Item | Cost |
|------|------|
| AWS WorkSpaces Power bundle (Ubuntu, hourly) | $7.25/month base + $0.66/hour |
| Anthropic API (Claude Code) | ~$2–10 per student per 4-hour session |

**Example — 10 students, 4-hour class:**
- WorkSpaces: $72.50 base + (40 hours × $0.66) ≈ **$99**
- API tokens: ~$50–100 (varies by usage)
- **Total: ~$150–200 per session**

---

## Part 1: One-Time Setup

### 1.1 Create AWS WorkSpaces Directory

1. Log in to AWS Console → WorkSpaces
2. Create a Simple AD directory (or use existing Active Directory)
3. Register the directory with WorkSpaces
4. Create an initial admin user

### 1.2 Launch a Template WorkSpace

1. Launch a single WorkSpace with:
   - **Bundle:** Ubuntu Power (4 vCPU, 16 GB RAM)
   - **Running mode:** AutoStop (hourly billing)
2. Wait for it to become AVAILABLE (~20 minutes)
3. Connect via the web client at https://clients.amazonworkspaces.com/webclient

### 1.3 Install Software on the Template

SSH in or connect via the web client, then run:

```bash
#!/bin/bash
# --- Docker ---
sudo apt-get update
sudo apt-get install -y docker.io docker-compose
sudo usermod -aG docker $USER

# --- Node.js (for Claude Code) ---
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# --- Claude Code ---
npm install -g @anthropic-ai/claude-code

# --- Clone and build EnergyPlus MCP ---
cd ~
git clone https://github.com/LBNL-ETA/EnergyPlus-MCP.git
cd EnergyPlus-MCP/.devcontainer
docker build -t energyplus-mcp-dev .

# --- Configure MCP for Claude Code ---
mkdir -p ~/.claude
cat > ~/.claude.json << 'EOF'
{
  "mcpServers": {
    "energyplus": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "-v", "/home/USER/EnergyPlus-MCP:/workspace",
        "-w", "/workspace/energyplus-mcp-server",
        "energyplus-mcp-dev",
        "uv", "run", "python", "-m", "energyplus_mcp_server.server"
      ]
    }
  }
}
EOF

# --- Create exercises directory ---
mkdir -p ~/exercises
```

> Replace `USER` in the volume mount path with the actual username.

### 1.4 Test the Installation

```bash
export ANTHROPIC_API_KEY="sk-ant-api03-your-key-here"
cd ~/exercises
claude
```

In Claude Code, type `/mcp` — you should see `energyplus: connected`.

### 1.5 Create a Custom Bundle (Recommended)

1. In AWS Console → WorkSpaces → Select your configured WorkSpace
2. Actions → **Create Image**
3. Name it: `EnergyPlus-MCP-Training-v1`
4. Wait for image creation (~30–60 minutes)
5. Go to **Bundles** → **Create Bundle** from your image
6. Now you can launch pre-configured WorkSpaces for students instantly

---

## Part 2: Per-Class Setup

### Launch Student WorkSpaces

1. Go to WorkSpaces → **Launch WorkSpaces**
2. Create users: `student01`, `student02`, etc.
3. Select your custom bundle
4. Launch all at once
5. Students receive email with login instructions

### API Key Options

**Option A — Shared key (simplest):**
Bake into the image or provide at class start:
```bash
echo 'export ANTHROPIC_API_KEY="sk-ant-api03-PROVIDED-BY-INSTRUCTOR"' >> ~/.bashrc
source ~/.bashrc
```

**Option B — Per-student keys:**
Create individual keys at https://console.anthropic.com and distribute to each student.

---

## Part 3: Student Quick Start Handout

Copy and distribute this to students:

---

### Getting Started

1. Open your browser and go to: **https://clients.amazonworkspaces.com/webclient**
2. Enter the **registration code** provided by your instructor
3. Log in with your **username and password**
4. Open a terminal (`Ctrl+Alt+T`)
5. If needed, set your API key:
   ```bash
   export ANTHROPIC_API_KEY="your-key-from-instructor"
   ```
6. Start Claude Code:
   ```bash
   cd ~/exercises
   claude
   ```
7. Verify MCP connection — type `/mcp` and confirm `energyplus: connected`
8. Try your first query:
   ```
   What EnergyPlus tools do you have available?
   ```

---

## Post-Class Cleanup

To minimize costs:

1. Set all student WorkSpaces to **AutoStop** (1-hour timeout)
2. After the class series ends, **terminate** student WorkSpaces
3. Keep your template WorkSpace for future classes
4. Monitor API usage at https://console.anthropic.com
