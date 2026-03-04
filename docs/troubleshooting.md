# Troubleshooting

Common issues encountered during EnergyPlus MCP setup and usage, based on real-world debugging.

---

## Docker Issues

### "Cannot connect to Docker daemon"
Docker Desktop must be fully running before starting Claude Desktop or Claude Code. Launch Docker Desktop and wait for the whale icon to stop animating before proceeding.

### "Image not found" when starting MCP server
The Docker image must be built locally — there is no pre-built image on Docker Hub.

```bash
cd EnergyPlus-MCP/.devcontainer
docker build -t energyplus-mcp-dev .
```

### Docker build fails or hangs
Check available disk space — the image is approximately 2 GB. Also verify your internet connection, as the build downloads EnergyPlus 25.1.0.

---

## Claude Desktop / MCP Connection Issues

### MCP tools don't appear in conversation
- MCP tools only load when you start a **new conversation**. They won't appear in existing chats.
- Confirm Docker Desktop is running.
- Completely quit and restart Claude Desktop (not just close the window — exit from system tray on Windows or menu bar on Mac).

### JSON syntax error in config
Common mistakes in `claude_desktop_config.json`:
- Missing comma between entries
- Trailing comma after last entry
- Smart quotes instead of straight quotes (happens if editing in Word or Notes)
- Unescaped backslashes in Windows paths

### Console output / JSON-RPC parsing errors
EnergyPlus simulation progress messages can leak into the MCP transport stream, causing JSON-RPC parsing errors in the console. **This is cosmetic** — your simulation results are still valid. The server continues to function normally.

---

## Path Issues (Windows)

### Hardcoded Linux paths in config.py
If running the server **outside** Docker on Windows, the `config.py` file may contain Linux-style paths like `/app/software/EnergyPlusV25-1-0`. These need to be updated:

```powershell
# PowerShell fix
(Get-Content energyplus_mcp_server\config.py) -replace '/app/software/EnergyPlusV25-1-0', 'C:/EnergyPlusV25-1-0' | Set-Content energyplus_mcp_server\config.py
```

> **Note:** If running via Docker (the recommended approach), the Linux paths are correct — don't change them.

### Windows backslash problems
Always use forward slashes in `claude_desktop_config.json`, even on Windows:

```
✅  C:/Users/YourName/EnergyPlus-MCP
❌  C:\Users\YourName\EnergyPlus-MCP
```

---

## EnergyPlus Version Issues

### Working with DOE Prototype Building Models
The official DOE prototype models from the Building Energy Codes Program are in EnergyPlus v22.1.0 format. The MCP server runs v25.1.0. You must update the IDF files before use.

**Option A — IDFVersionUpdater GUI (local EnergyPlus install required):**
1. Install EnergyPlus locally (any recent version)
2. Open the IDFVersionUpdater application
3. Ensure the "E+ Folder" points to your EnergyPlus installation directory (e.g., `/Applications/EnergyPlus-25-2-0`), **not** to your model files
4. Load your v22 IDF file and run the update chain

**Option B — Command-line transitions (in Docker):**
The transition executables chain from one version to the next. Each step must be run sequentially (v22.1→v22.2→v23.1→...→v25.1).

---

## "dot_parser" Import Warning
```
Couldn't import dot_parser, loading of dot files will not be possible.
```
This warning is harmless and does not affect MCP server functionality. It relates to the `pydot` library used for graph visualization and can be safely ignored.
