---
name: nexus:status
description: Check Nexus VPS services (docker, syncthing, claudius)
---

# Nexus Status Check

Check the health of services running on the Nexus VPS.

## Determine context

First, check if we're running on Nexus or remotely:

```bash
hostname
```

- If `nexus`: run commands directly
- Otherwise: run via `ssh nexus` (requires Tailscale)

## Checks

Run these commands (directly or via ssh) and report results:

1. **System**: `uptime` and `df -h /`
2. **Docker**: `docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'`
3. **Syncthing**: `systemctl --user status syncthing | head -5`
4. **Claudius**: `docker ps -f name=claudius --format '{{.Status}}'` (if running)
5. **Tailscale**: `tailscale status --peers=false`

## Output

Present as a status dashboard:

```
Nexus VPS Status
================
System:    <uptime> | <disk usage>
Docker:    <N containers running>
Syncthing: <active/inactive>
Claudius:  <running/stopped>
Tailscale: <connected/disconnected>
```

List any services that are down or unhealthy as warnings.
