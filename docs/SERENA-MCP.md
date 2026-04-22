# Serena-MCP on Unraid

Run Serena on Unraid using Docker. Serena is a powerful MCP (Model Context Protocol) server that gives AI coding agents IDE-level code intelligence using language servers (LSP).

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Docker Template Setup](#docker-template-setup)
- [First Start Setup](#first-start-setup)
- [Mounting Projects](#mounting-projects)
- [Connecting an MCP Client](#connecting-an-mcp-client)
- [Web Dashboard](#web-dashboard)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

- Unraid 6.12 or later
- An MCP-compatible client (Claude Code, Claude Desktop, etc.)
- Projects accessible from Unraid (on the array, cache, or mounted network share)

> **Tested on:** Unraid 6.12+ · Serena 1.26.0

---

## Docker Template Setup

### Community Applications (Recommended)

1. Open the **Apps** tab in Unraid and search for **Serena-MCP**
2. Click **Install**
3. Configure paths (see below) and click **Apply**

### Manual Template

Copy the template to your flash drive:
```
/boot/config/plugins/dockerMan/templates-user/serena_mcp.xml
```

#### Basic Settings

| Field | Value |
|-------|-------|
| **Name** | `Serena-MCP` |
| **Repository** | `ghcr.io/oraios/serena` |
| **Network Type** | `bridge` |
| **MCP Server Port** | `9121` |
| **Dashboard Port** | `24282` |

---

## First Start Setup

On first start, Serena generates `serena_config.yml` with the dashboard bound to `127.0.0.1`, which makes it inaccessible from outside the container. This is a one-time fix.

1. Start the container and let it fully initialize
2. Edit `/mnt/user/appdata/serena-mcp/config/serena_config.yml`
3. Make the following changes:

```yaml
web_dashboard_listen_address: 0.0.0.0
web_dashboard_open_on_launch: False
gui_log_window: False
```

4. Restart the container

The dashboard will now be accessible at `http://[SERVER_IP]:24282`.

---

## Mounting Projects

Serena can only access directories that are mounted into the container. Each project must be added as its own volume mount.

### Projects on Unraid

In the Unraid template UI, click **Add another Path** for each project:

| Field | Value |
|-------|-------|
| **Host path** | `/mnt/user/your-project/` |
| **Container path** | `/workspace/your-project/` |

### Projects on your Personal Computer

If your projects live on a PC or Mac rather than on Unraid, you need to share and mount them first:

1. **Share the folder** from your PC/Mac over SMB (Windows share) or NFS
2. On Unraid, install the **Unassigned Devices** plugin
3. In Unassigned Devices, mount the network share — it will appear at a path like `/mnt/remotes/my-pc-share/`
4. Add the mount in the Serena template:

| Field | Value |
|-------|-------|
| **Host path** | `/mnt/remotes/my-pc-share/your-project/` |
| **Container path** | `/workspace/your-project/` |

Your project files stay on your PC but Serena can read and index them.

### Registering a Project

Once mounted, you need to tell Serena about the project. The easiest way is via your MCP client — ask it to:

```
activate the project at /workspace/your-project/
```

Or use the Serena dashboard to register it manually.

---

## Connecting an MCP Client

### Claude Code

Add the following to your Claude Code MCP config:

```json
{
  "mcpServers": {
    "serena": {
      "type": "http",
      "url": "http://[SERVER_IP]:9121/mcp"
    }
  }
}
```

### Other MCP Clients

Connect to:
```
http://[SERVER_IP]:9121/mcp
```

Transport: `streamable-http`

---

## Web Dashboard

The Serena dashboard provides an overview of the active project, registered projects, tool usage, and configuration.

Access it at: `http://[SERVER_IP]:24282`

> The dashboard requires the one-time config change described in [First Start Setup](#first-start-setup).

---

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `SERENA_DOCKER` | `1` | Enables Docker-specific behavior. Keep set to `1`. |
| `INTELEPHENSE_LICENSE_KEY` | *(empty)* | Optional license key for Intelephense PHP language server premium features. |

---

## Troubleshooting

**Container starts but immediately stops**
Ensure PostArgs are set correctly. The container requires an explicit startup command.

**Dashboard not accessible at port 24282**
Complete the [First Start Setup](#first-start-setup) — the dashboard binds to `127.0.0.1` by default and must be changed to `0.0.0.0`.

**MCP client gets 406 error**
Ensure you are sending the correct headers. The endpoint requires:
- `Content-Type: application/json`
- `Accept: application/json, text/event-stream`

**Project not visible in Serena**
The project must be both mounted as a volume AND registered with Serena. See [Registering a Project](#registering-a-project).

---

## Links

- [Serena GitHub](https://github.com/oraios/serena)
- [Serena Docker Docs](https://github.com/oraios/serena/blob/main/DOCKER.md)
- [Template Repository](https://github.com/PikkonMG/unraid-docker-templates)
