# Nexus Orchestrator on Unraid

Run Nexus Orchestrator on Unraid using Docker. This guide covers installation, configuration, first-time setup, and how to use the app.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Docker Template Setup](#docker-template-setup)
- [Environment Variables](#environment-variables)
- [First Run](#first-run)
- [First-Time Setup](#first-time-setup)
- [Using the App](#using-the-app)
- [Remote Access](#remote-access)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

- Unraid 6.12 or later
- [Ollama](https://ollama.com) running on your network (local machine, NAS, or another server) with at least one model pulled
- The Ollama host IP — **do not use `localhost`** inside Docker containers

> **Tested on:** Unraid 6.12+ · Nexus Orchestrator 1.0.9 · Ollama 0.6+

---

## Docker Template Setup

### Community Applications (Recommended)

1. Open the **Apps** tab in Unraid and search for **Nexus Orchestrator**
2. Click **Install**
3. Fill in the environment variables (see below) and click **Apply**

### Manual Template

If the CA template is not yet available, click **Add Container** in the Unraid Docker tab and fill in the following fields.
Or
Copy the template to your flash drive: /boot/config/plugins/dockerMan/templates-user/nexus-orchestrator.xml

#### Basic Settings

| Field | Value |
|-------|-------|
| **Name** | `nexus-orchestrator` |
| **Repository** | `pikkonmg/nexus-orchestrator:latest` |
| **Network Type** | `bridge` |

#### Port Mapping

| Container Port | Host Port | Protocol |
|---------------|-----------|----------|
| `3000` | `3000` | `TCP` |

Change the host port if `3000` is already in use on your server.

#### Volume Mapping

| Container Path | Host Path | Access Mode |
|---------------|-----------|-------------|
| `/app/data` | `/mnt/user/appdata/nexus-orchestrator` | Read/Write |

This is where your database, config, and conversation history are stored. All data persists across container restarts and updates.

---

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `ADMIN_API_KEY` | ✅ Yes | — | Password for the web UI and API. Set a strong value. |
| `ENCRYPTION_SECRET` | Recommended | *(derived from ADMIN_API_KEY)* | Separate secret for encrypting stored data. Set a unique value in production. |
| `LOCAL_URL` | ✅ Yes | `http://localhost:11434` | Base URL of your Ollama (or local OpenAI-compatible) provider. Use the LAN IP — not `localhost`. Example: `http://192.168.1.50:11434` |
| `LOCAL_KEY` | No | *(empty)* | API key for local provider. Leave blank for standard Ollama. |
| `CLOUD_URL` | No | *(empty)* | Base URL for an optional cloud provider (e.g. `https://api.openai.com/v1`). Leave blank for fully local operation. |
| `CLOUD_API_KEY` | No | *(empty)* | API key for the cloud provider. |
| `ROUTER_MODEL` | ✅ Yes | *(empty)* | The model used to classify prompts. A small fast model works well (e.g. `gemma3:4b`, `qwen2.5:3b`). Must be available on your provider. |
| `ROUTER_URL` | No | *(same as LOCAL_URL)* | Custom URL for the intent router. Leave blank to use your Local Provider. |
| `ROUTER_KEY` | No | *(empty)* | API key for the router endpoint if different from local/cloud. |
| `PORT` | No | `3000` | Internal server port. Only change if you need a non-standard internal port. |
| `CONFIG_DIR` | No | `/app/data` | Where config and database are stored inside the container. Leave as default. |
| `LOG_LEVEL` | No | `info` | Pino log verbosity: `trace`, `debug`, `info`, `warn`, `error`. |

### Minimal working config

```
ADMIN_API_KEY=changeme_use_a_strong_key
LOCAL_URL=http://192.168.1.50:11434
ROUTER_MODEL=gemma3:4b
```

### Full local + cloud hybrid example

```
ADMIN_API_KEY=changeme_use_a_strong_key
ENCRYPTION_SECRET=another_unique_secret
LOCAL_URL=http://192.168.1.50:11434
ROUTER_MODEL=gemma3:4b
CLOUD_URL=https://api.openai.com/v1
CLOUD_API_KEY=sk-...
```

---

## First Run

Start the container and watch logs in the Unraid Docker UI or via terminal:

```bash
docker logs -f nexus-orchestrator
```

A successful startup looks like:

```
{"level":"info","msg":"Nexus Orchestrator listening on port 3000"}
```

Open `http://[unraid-ip]:3000` in your browser and log in with your `ADMIN_API_KEY`.

---

## First-Time Setup

After logging in, complete these steps in the **Models** tab before using the chat.

### 1. Verify Local Provider connection

The header bar shows a green **Online** indicator when Nexus can reach your Ollama instance. If it shows red, check your `LOCAL_URL` — make sure it's the LAN IP, not `localhost`.

### 2. Discover models

Click **Refresh** next to Discovered Models. All models available on your Ollama instance will appear as clickable chips.

### 3. Assign models to categories

Each category (CODING, REASONING, CREATIVE, etc.) has a model pool. Click a discovered model chip to add it to a category.

- The **first model** in the pool is used by default
- If it fails, Nexus automatically tries the next model in the pool
- You can type a custom model name if it doesn't appear in the list

**Recommended starter setup:**

| Category | Suggested model type |
|----------|---------------------|
| GENERAL | A mid-size general model (e.g. `llama3.2:3b`, `mistral:7b`) |
| CODING | A code-focused model (e.g. `qwen2.5-coder:7b`, `deepseek-coder:6.7b`) |
| REASONING | A larger or reasoning-focused model (e.g. `deepseek-r1:8b`) |
| CREATIVE | Any general model — or the same as GENERAL |
| VISION | A vision-capable model (e.g. `llava:7b`, `minicpm-v:8b`) |
| DOCUMENT | Same as VISION or a general model |
| FAST | Your smallest/fastest model (e.g. `gemma3:1b`, `qwen2.5:1.5b`) |
| SECURITY | A capable general or reasoning model |

### 4. Set your router model

The router is the small model that reads each prompt and decides which category to route it to. It only needs to return clean JSON — a 1–4B parameter model is ideal.

- Go to **Models** tab → **Intent Router** section
- The Router Model is set via the `ROUTER_MODEL` env var at container startup
- If you want to override it without restarting, you can set it in the UI — but env var takes priority on restart

---

## Using the App

### Sending a message

Type your message in the chat input and press **Enter** (or **Shift+Enter** for a new line). Nexus routes the prompt to the best category automatically. Each response shows which router model made the routing decision.

### Attaching files

- **Images** — click the attachment button and select an image. Nexus forces the request to the VISION category and sends the image in the correct format for Ollama vision models.
- **Documents** (PDF, text) — attach a document to force the DOCUMENT category.

### Stop generation

Click the red stop button to abort a response mid-stream. The partial response is kept in chat. Note: Ollama will continue generating in the background briefly — this is a known Docker networking limitation.

### Conversations

- New conversations are created automatically on first message
- Double-click a conversation title in the sidebar to rename it
- Right-click a conversation for options: **Rename**, **Move to Project**, **Delete**

### Projects

Organize conversations into named project folders:

1. Click **+ New Project** in the sidebar
2. Enter a project name
3. Right-click any conversation → **Move to Project** → select the project
4. Click a project header to collapse or expand it
5. Double-click a project name to rename it
6. Click the trash icon on a project to delete it — you can choose to keep the chats (they move to unassigned) or delete them all

### Router Result Caching

By default, every message makes a fresh call to the router model. If you're using a **paid cloud router**, you can enable caching to avoid duplicate API calls:

1. Go to the **System** tab
2. Toggle **Router Result Caching** on

Identical prompts reuse the cached decision for 5 minutes. The cache is in-memory and clears on container restart.

### Exporting data

In the **System** tab:

- **Export JSON** under Configuration — downloads your current config
- **Export JSON** under Conversations — downloads all conversations with full message history

---

## Remote Access

If Nexus runs in Docker but your Ollama is on a different machine (e.g. your desktop), expose Ollama via a tunnel for remote access:

```bash
# Cloudflare tunnel
cloudflared tunnel --url http://localhost:11434

# Ngrok
ngrok http 11434
```

Then set `LOCAL_URL` to the tunnel address.

---

## Troubleshooting

### Provider shows Offline / red indicator

**Cause:** Nexus cannot reach the URL in `LOCAL_URL`.

**Fix:** Make sure you're using the LAN IP of the machine running Ollama, not `localhost`. Inside Docker, `localhost` refers to the container itself.

```
# Wrong
LOCAL_URL=http://localhost:11434

# Right
LOCAL_URL=http://192.168.1.50:11434
```

---

### No models appear in Discovered Models

**Cause:** Ollama is unreachable, or the URL is wrong.

**Fix:**
1. Verify the provider shows **Online** in the header
2. Confirm Ollama is running: `curl http://192.168.1.50:11434/api/tags`
3. Check that Ollama is bound to `0.0.0.0` and not just `127.0.0.1`

To make Ollama listen on all interfaces, set the environment variable on your Ollama host:

```bash
OLLAMA_HOST=0.0.0.0 ollama serve
```

Or if running as a service, add it to the systemd unit.

---

### Router not routing correctly / returning wrong category

**Cause:** The router model may not support JSON output well, or it's too small.

**Fix:**
- Try a slightly larger model (3B+ works well — `gemma3:4b`, `qwen2.5:3b`)
- Nexus automatically strips markdown code fences from the router response, so models that wrap JSON in ` ```json ``` ` blocks are fine
- Check the **Routing Analysis** section below each response to see exactly what the router decided

---

### Chat responses time out on first message to a large model

**Cause:** Ollama is loading the model into VRAM for the first time. This can take 10–60+ seconds for large models.

**Fix:** This is expected. Nexus has a 60-second per-attempt timeout and will retry up to 3 times. Wait for the model to load — subsequent messages will be fast.

---

### Conversation or config changes not persisting after restart

**Cause:** The `/app/data` volume is not mapped, or the host path doesn't have write permissions.

**Fix:** Verify the volume mapping in your container config points to a writable path on your Unraid array:

```
Container path:  /app/data
Host path:       /mnt/user/appdata/nexus-orchestrator
```

---

### General error reference

| Symptom | Cause | Fix |
|---------|-------|-----|
| Provider shows red / Offline | Wrong `LOCAL_URL` | Use LAN IP, not `localhost` |
| No models in Discovered Models | Ollama unreachable or not listening on `0.0.0.0` | Check Ollama host binding |
| Router returns invalid JSON | Router model too small or not JSON-capable | Use a 3B+ model for routing |
| First message times out | Model loading into VRAM | Wait for retry — normal on cold start |
| Settings not saving | `/app/data` not writable | Check volume mapping and permissions |
| UI shows blank tab after error | React render error | Error boundary caught it — click "Try again" in the tab |

---

## Resources

- [Nexus Orchestrator on GitHub](https://github.com/FaqFirebase/Nexus-Orchestrator)
- [Nexus Orchestrator on Docker Hub](https://hub.docker.com/r/pikkonmg/nexus-orchestrator)
- [Ollama Documentation](https://ollama.com/docs)
- [Unraid Docker Documentation](https://docs.unraid.net/unraid-os/manual/docker-management/)
