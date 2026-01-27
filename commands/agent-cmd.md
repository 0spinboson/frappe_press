---
name: agent-cmd
description: Run commands on Press servers via the agent API - execute commands in containers or on host
argument-hint: <server> <command> [--bench <bench-name>]
allowed-tools: ["Bash", "Read"]
---

# Agent Command Execution

Run commands on Press servers via the agent HTTP API. Can execute commands inside Docker containers (benches) or on the host system.

## Setup

```python
/home/frappe/press/env/bin/python << 'PYEOF'
import frappe
import requests
frappe.init(site='veg10.veganisme.org', sites_path='/home/frappe/press/sites')
frappe.connect()

# Get server and agent credentials
server = frappe.get_doc("Server", "SERVER_NAME_HERE")
agent_password = server.get_password("agent_password")

# Agent API base URL
agent_url = f"https://{server.name}/agent"
auth = ("frappe", agent_password)

# Your API calls here

frappe.destroy()
PYEOF
```

## Agent API Endpoints

### Ping Agent
```python
response = requests.get(f"{agent_url}/ping", auth=auth, verify=True)
print(response.json())  # {"message": "pong"}
```

### List Benches on Server
```python
response = requests.get(f"{agent_url}/benches", auth=auth, verify=True)
benches = response.json()
for bench in benches:
    print(bench)
```

### Execute Command in Bench Container
```python
bench_name = "bench-0004-000055-hybrid"
command = "ls -la sites/assets/"

response = requests.post(
    f"{agent_url}/benches/{bench_name}/command",
    auth=auth,
    json={"command": command},
    verify=True,
    timeout=30
)
result = response.json()
print(result.get("output", result))
```

### Execute Command on Host
```python
# Run command on host system (not in container)
command = "docker ps --format '{{.Names}}'"

response = requests.post(
    f"{agent_url}/run",
    auth=auth,
    json={"command": command},
    verify=True,
    timeout=30
)
print(response.json())
```

## Common Commands

### Check Assets in Container
```bash
# List assets directory
command = "ls -la sites/assets/"

# Check specific app assets
command = "ls -la sites/assets/verenigingen/"

# Check if symlink or directory
command = "file sites/assets/verenigingen"
```

### Check Container Status
```bash
# On host - list running containers
command = "docker ps --format '{{.Names}}\t{{.Status}}'"

# Check container logs
command = "docker logs bench-0004-000055-hybrid --tail 50"
```

### Check Site Files
```bash
# In container - list site directory
command = "ls -la sites/mijnrsp.veganisme.net/"

# Check lock files
command = "ls -la sites/mijnrsp.veganisme.net/locks/"

# Check site config
command = "cat sites/mijnrsp.veganisme.net/site_config.json"
```

### Bench Operations
```bash
# Check scheduler status
command = "bench doctor"

# Check app versions
command = "bench version"

# List installed apps
command = "cat sites/apps.txt"
```

## Error Handling

The agent may return errors in different formats:

```python
result = response.json()
if "error" in result:
    print(f"Error: {result['error']}")
elif "output" in result:
    print(result["output"])
else:
    print(result)
```

## Argument Parsing

- `<server>` - Server name (e.g., "hybrid.veganisme.net") or partial match
- `<command>` - Command to execute (quote if contains spaces)
- `--bench <name>` or `-b <name>` - Execute in specific bench container
- `--host` or `-H` - Execute on host instead of in container

## Finding Server by Partial Name

```python
# Find server by partial name
partial = "hybrid"
servers = frappe.get_all("Server",
    filters={"name": ["like", f"%{partial}%"]},
    fields=["name", "ip"])
if servers:
    server_name = servers[0].name
```

**User's request:** $ARGUMENTS
