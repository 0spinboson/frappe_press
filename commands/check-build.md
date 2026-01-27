---
name: check-build
description: Investigate a deploy candidate build - show status, apps, build steps, errors, and search build output
argument-hint: <build-tag> [--output] [--errors] [--search <term>]
allowed-tools: ["Bash", "Read"]
---

# Check Deploy Candidate Build

Investigate a deploy candidate build to understand its status, what apps were included, build steps, and any errors.

## Setup

```python
/home/frappe/press/env/bin/python << 'PYEOF'
import frappe
frappe.init(site='veg10.veganisme.org', sites_path='/home/frappe/press/sites')
frappe.connect()

# Query code here

frappe.destroy()
PYEOF
```

## Build Investigation

### Get Build Overview
```python
build_tag = "BUILD_TAG_HERE"
build = frappe.get_doc("Deploy Candidate Build", build_tag)

print(f"Build: {build.name}")
print(f"Status: {build.status}")
print(f"Docker Image: {build.docker_image}")
print(f"Deploy Candidate: {build.deploy_candidate}")

# Get apps from deploy candidate
dc = frappe.get_doc("Deploy Candidate", build.deploy_candidate)
print(f"\nApps ({len(dc.apps)}):")
for app in dc.apps:
    print(f"  - {app.app}")
```

### Check Build Steps
```python
# Show build steps with non-success status
for step in build.build_steps:
    if step.status != "Success":
        print(f"Step: {step.stage} - {step.step}")
        print(f"Status: {step.status}")
        if step.output:
            print(f"Output: {step.output[:500]}")
```

### Get Build Output
```python
# Show last N characters of build output
if build.build_output:
    print("--- Build Output (last 3000 chars) ---")
    print(build.build_output[-3000:])
```

### Search Build Output
```python
# Search for a term in build output
search_term = "SEARCH_TERM"
if build.build_output:
    lines = build.build_output.split('\n')
    for i, line in enumerate(lines):
        if search_term.lower() in line.lower():
            start = max(0, i-2)
            end = min(len(lines), i+3)
            print(f"--- Found at line {i} ---")
            for j in range(start, end):
                marker = ">>>" if j == i else "   "
                print(f"{marker} {lines[j]}")
            print()
```

### Find Errors in Build Output
```python
# Look for common error patterns
error_patterns = ["error", "failed", "Error:", "FAILED", "exit code", "Exception"]
if build.build_output:
    lines = build.build_output.split('\n')
    for i, line in enumerate(lines):
        if any(pat in line for pat in error_patterns):
            print(f"Line {i}: {line[:200]}")
```

## Argument Parsing

- `<build-tag>` - The build tag/name (e.g., "6lrg7mqd5m")
- `--output` or `-o` - Show full build output
- `--errors` or `-e` - Search for errors in output
- `--search <term>` or `-s <term>` - Search build output for a term
- `--latest <bench-prefix>` - Get the latest build for a bench (e.g., "bench-0004")

## Finding Latest Build

```python
# Find latest build for a bench prefix
bench_prefix = "bench-0004"
builds = frappe.get_all("Deploy Candidate Build",
    filters={"docker_image": ["like", f"%{bench_prefix}%"]},
    fields=["name", "status", "docker_image", "creation"],
    order_by="creation desc",
    limit=1)
if builds:
    build_tag = builds[0].name
```

**User's request:** $ARGUMENTS
