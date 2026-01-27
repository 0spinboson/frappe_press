---
name: check-site
description: Check site status, bench, server, migration status, and diagnose common issues like asset accessibility
argument-hint: <site-name> [--assets] [--jobs] [--migrations]
allowed-tools: ["Bash", "Read", "WebFetch"]
---

# Check Site Status

Investigate a site to understand its status, which bench and server it's on, recent agent jobs, and diagnose common issues.

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

## Site Investigation

### Get Site Overview
```python
site_name = "SITE_NAME_HERE"
site = frappe.get_doc("Site", site_name)

print(f"Site: {site.name}")
print(f"Status: {site.status}")
print(f"Bench: {site.bench}")
print(f"Server: {site.server}")
print(f"Plan: {site.plan}")

# Get bench details
bench = frappe.get_doc("Bench", site.bench)
print(f"\nBench Status: {bench.status}")
print(f"Docker Image: {bench.docker_image}")
```

### Check Recent Agent Jobs
```python
# Get recent agent jobs for this site
jobs = frappe.get_all("Agent Job",
    filters={"site": site_name},
    fields=["name", "job_type", "status", "creation"],
    order_by="creation desc",
    limit=10)

print("\nRecent Agent Jobs:")
for job in jobs:
    print(f"  {job.creation} | {job.job_type} | {job.status}")
```

### Check Migration Status
```python
# Check for recent migrate jobs
migrate_jobs = frappe.get_all("Agent Job",
    filters={"site": site_name, "job_type": "Migrate Site"},
    fields=["name", "status", "creation", "output"],
    order_by="creation desc",
    limit=3)

for job in migrate_jobs:
    print(f"\nMigration {job.name}: {job.status}")
    if job.status == "Failure" and job.output:
        print(f"Error: {job.output[-500:]}")
```

## Asset Accessibility Check

Use curl to check if assets are accessible:

```bash
# Check if a specific asset is accessible
curl -sI "https://SITE_NAME/assets/frappe/dist/css/website.bundle.HASH.css" | head -10

# Check if an app's assets folder exists
curl -sI "https://SITE_NAME/assets/APP_NAME/" | head -5
```

Common asset issues:
- 404 on `/assets/<app>/` - App assets not linked in Docker image
- 404 on specific file - Build failed or file not generated

## Server and Container Info

```python
# Get server details
server = frappe.get_doc("Server", site.server)
print(f"\nServer: {server.name}")
print(f"IP: {server.ip}")
print(f"Status: {server.status}")
```

## Diagnose Common Issues

### Site Not Loading
1. Check site status (should be "Active")
2. Check bench status (should be "Active")
3. Check recent agent jobs for failures

### Migration Failures
1. Look for "Migrate Site" jobs with "Failure" status
2. Check job output for specific errors
3. Common issues: DocumentLockedError, QueueOverloaded

### Assets Not Loading
1. Check if asset path returns 404
2. If app assets missing, likely a Docker build issue
3. Check build output for that app's asset linking

## Argument Parsing

- `<site-name>` - The site name (e.g., "mijnrsp.veganisme.net")
- `--assets` or `-a` - Check asset accessibility
- `--jobs` or `-j` - Show recent agent jobs
- `--migrations` or `-m` - Focus on migration status

**User's request:** $ARGUMENTS
