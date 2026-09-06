# Hindsight on Unraid

Shared memory for AI agents, reachable over MCP and HTTP. This template runs
the official image `ghcr.io/vectorize-io/hindsight:latest`, which bundles the
API, the web dashboard, the local search models, and an embedded PostgreSQL
database. Hindsight also needs one language model, which you point at a local
Ollama server or a hosted API.

## First start

1. Make the data folders on the Unraid terminal before you start the
   container:

   ```bash
   mkdir -p /mnt/user/appdata/hindsight
   chown -R 1000:1000 /mnt/user/appdata/hindsight
   chmod 700 /mnt/user/appdata/hindsight
   ```

   The container runs as user 1000 and does not read Unraid's PUID and PGID
   settings. If you change a path in the template, use the same path here.

2. Copy `templates/hindsight.xml` to
   `/boot/config/plugins/dockerMan/templates-user/my-hindsight.xml` on
   Unraid. Then open **Docker > Add Container** and pick **hindsight** from
   the user templates.

3. Set the model fields. The template defaults to local Ollama:

   | Field | Value |
   |---|---|
   | Model service | `ollama` |
   | Model name | `gemma4:12b` |
   | Model endpoint | `http://YOUR_UNRAID_IP:11434/v1` |
   | Model provider key | empty |

   The model must support tool calling. Pull it on the Ollama host first:

   ```bash
   ollama pull gemma4:12b
   ```

   For any other service, use these values:

   | Model service | Provider | Model | Endpoint | Provider key |
   |---|---|---|---|---|
   | Local Ollama | `ollama` | Exact local name | `http://YOUR_UNRAID_IP:11434/v1` | Blank |
   | OpenAI API key | `openai` | `gpt-4o-mini` | Blank | OpenAI key |
   | Anthropic API key | `anthropic` | `claude-haiku-4-5` | Blank | Anthropic key |
   | Google Gemini API key | `gemini` | `gemini-3.5-flash` | Blank | Gemini key |
   | Groq API key | `groq` | `openai/gpt-oss-120b` | Blank | Groq key |
   | Ollama Cloud key | `ollama-cloud` | `gpt-oss:120b` | Blank | Ollama key |
   | OpenRouter key | `openrouter` | Exact OpenRouter model id | Blank | OpenRouter key |
   | DeepSeek key | `deepseek` | `deepseek-v4-flash` | Blank | DeepSeek key |
   | xAI Grok API key | `openai` | Exact xAI model name | `https://api.x.ai/v1` | xAI key |
   | Other OpenAI-compatible API | `openai` | Exact API name | Provider URL ending in `/v1` | API key |

   **Model service** also accepts these:

   `openai-responses`, `vertexai`, `lmstudio`, `llamacpp`, `minimax`, `zai`,
   `opencode-go`, `atlas`, `meta`, `volcano`, `requesty`, `github-copilot`,
   `bedrock`, `fireworks`, `nous`, `litellm`, `litellmrouter`

   The [Hindsight models documentation](https://hindsight.vectorize.io/developer/models)
   lists the extra variables some of them need.

4. Set the three keys. **Memory API key** and **Web UI backend key** must
   hold the same value. **Web UI login key** is a separate one.

5. Apply the template. Open `http://YOUR_UNRAID_IP:9999/` and sign in with
   the Web UI login key. Leave the internal Web UI backend URL at
   `http://127.0.0.1:8888`.

## Connect agents

Upstream ships an installer that detects the coding agents you already have
and wires them all up. Run it on the machine you code from, not on Unraid.
This has only been run on Linux.

```bash
npx @vectorize-io/hindsight-coding-agents install all \
  --server self-hosted \
  --api-url http://YOUR_UNRAID_IP:8888 \
  --api-token YOUR_MEMORY_API_KEY
```

It covers Claude Code, Codex, Cursor, opencode and Pi, and writes one config
file at `~/.hindsight/coding-agent.json`. After that each repo gets its own
bank, seeded from its commit history.

A repo named `myproject` lands in a bank named `coding-agent::myproject`. To
send a folder to a bank you named yourself, add a mapping to that same file:

```json
{
  "mapPathToBank": {
    "/path/to/your/repo": "your-bank"
  }
}
```

### Anything the installer does not cover

It is a plain MCP endpoint over Streamable HTTP. One bank per project, named
in the URL:

- `http://YOUR_UNRAID_IP:8888/mcp/YOUR_BANK/`

Send the header `Authorization: Bearer YOUR_MEMORY_API_KEY`, using the
Memory API key from the template. Most tools read the same JSON shape:

```json
{
  "mcpServers": {
    "hindsight-YOUR_BANK": {
      "type": "http",
      "url": "http://YOUR_UNRAID_IP:8888/mcp/YOUR_BANK/",
      "headers": { "Authorization": "Bearer YOUR_MEMORY_API_KEY" }
    }
  }
}
```

| Tool | Where to put it |
|---|---|
| Cursor | `~/.cursor/mcp.json` for all projects, or `.cursor/mcp.json` in one project. Cursor ignores the `type` field. |
| Oh My Pi | `~/.omp/agent/mcp.json`. It expands `${HINDSIGHT_API_KEY}` in headers. |
| Plain Pi | Install the `pi-mcp` extension, then use `~/.pi/agent/mcp.json` with the same shape. |
| Grok CLI | Run `grok mcp add --transport http hindsight-YOUR_BANK http://YOUR_UNRAID_IP:8888/mcp/YOUR_BANK/ --header "Authorization: Bearer YOUR_MEMORY_API_KEY"` |
| Claude Desktop | Its connector dialog has nowhere for a bearer token, so use the `mcp-remote` bridge in `claude_desktop_config.json`: `"command": "npx", "args": ["mcp-remote", "http://YOUR_UNRAID_IP:8888/mcp/YOUR_BANK/", "--header", "Authorization: Bearer YOUR_MEMORY_API_KEY"]` |
| ChatGPT Desktop | Needs a public HTTPS server with OAuth or no auth, so a LAN address will not work. |

The shared API key opens every bank, so only give it to clients you trust.

## Storage and updates

The appdata path maps to `/home/hindsight/.pg0` inside the container. Keep
that mapping on updates. Before an update, stop the container and back up
the whole mapped folder.

To move to an external PostgreSQL with pgvector, export and import your
memories first, then change **Database URL** under **Show more settings**
from `pg0` to the connection string. Changing the URL alone does not move
existing memory. The external database must already exist with its vector
extension enabled before Hindsight starts.

## Sources

- [Docker setup and storage](https://hindsight.vectorize.io/developer/installation)
- [Model configuration and supported services](https://hindsight.vectorize.io/developer/models)
- [API and dashboard keys](https://hindsight.vectorize.io/developer/configuration)
- [MCP connections](https://hindsight.vectorize.io/developer/mcp-server)
- [Releases](https://github.com/vectorize-io/hindsight/releases)
