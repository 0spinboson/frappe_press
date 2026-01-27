---
name: press-query
description: Query the Frappe Press database for benches, sites, deploy candidates, builds, servers, and other records
argument-hint: <doctype> [filters] - e.g., "benches on hybrid", "site mijnrsp.veganisme.net", "build 6lrg7mqd5m"
allowed-tools: ["Bash", "Read"]
---

# Press Database Query

Query the Frappe Press database to retrieve information about benches, sites, deploy candidates, builds, servers, and other Press doctypes.

## Setup

The Press site is `veg10.veganisme.org` and the sites path is `/home/frappe/press/sites`.

Always use this Python boilerplate to connect:

```python
/home/frappe/press/env/bin/python << 'PYEOF'
import frappe
frappe.init(site='veg10.veganisme.org', sites_path='/home/frappe/press/sites')
frappe.connect()

# Your query here

frappe.destroy()
PYEOF
```

## Common Queries

### Benches
```python
# List recent benches for a server
benches = frappe.get_all("Bench",
    filters={"server": ["like", "%hybrid%"]},
    fields=["name", "status", "docker_image"],
    order_by="creation desc",
    limit_page_length=5)

# Get bench details
bench = frappe.get_doc("Bench", "bench-0004-000055-hybrid")
```

### Sites
```python
# Find a site
site = frappe.get_doc("Site", "mijnrsp.veganisme.net")
print(f"Bench: {site.bench}, Status: {site.status}")

# List sites on a bench
sites = frappe.get_all("Site",
    filters={"bench": "bench-0004-000055-hybrid"},
    fields=["name", "status"])
```

### Deploy Candidates & Builds
```python
# Get deploy candidate build
build = frappe.get_doc("Deploy Candidate Build", "6lrg7mqd5m")
print(f"Status: {build.status}")
if build.build_output:
    print(build.build_output[-2000:])  # Last 2000 chars

# Get deploy candidate apps
dc = frappe.get_doc("Deploy Candidate", build.deploy_candidate)
for app in dc.apps:
    print(f"  {app.app}")
```

### Servers
```python
# Get server info
server = frappe.get_doc("Server", "hybrid.veganisme.net")
print(f"IP: {server.ip}")
```

## Argument Parsing

Parse the user's argument to determine what to query:

- `benches on <server>` - List benches on a server
- `bench <name>` - Get bench details
- `site <name>` - Get site details
- `sites on <bench>` - List sites on a bench
- `build <tag>` - Get build details and output
- `builds for <bench-prefix>` - List recent builds
- `server <name>` - Get server details

## Output Format

Present results in a clear, readable format. For lists, use tables. For single records, show key fields.

**User's query:** $ARGUMENTS
