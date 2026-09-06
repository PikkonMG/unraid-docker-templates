# Unraid Docker Templates

A collection of Docker templates designed for **Unraid**.  
These templates make it easier to deploy advanced self-hosted tools with minimal configuration.

This repository focuses on providing **well-structured templates and documentation** so users can quickly deploy services through the Unraid GUI.

---

## Available Templates

### Hindsight

Shared long-term memory for AI agents, reachable over MCP from any tool.

📄 Documentation:  
➡️ [Hindsight Guide](docs/HINDSIGHT.md)

This template runs the **API, web dashboard, and embedded PostgreSQL** in one container, plus a separate AI model that turns saved text into searchable facts.

Features:

- Local Ollama by default, or a hosted API key service
- A separate memory bank for each project
- Shared by Claude Code, Cursor, Pi, Grok CLI, and Claude Desktop
- Data kept under appdata, with no project folder mounts
- API key and web dashboard login required

---

### NVIDIA NIM (Single Model)

Run NVIDIA NIM containers on Unraid to serve optimized AI inference models.

📄 Documentation:  
➡️ [NVIDIA NIM Single Template Guide](docs/NVIDIA-NIM_SINGLE.md)

This template allows you to run a **single NVIDIA NIM model container** on Unraid with GPU acceleration.

Features:

- GPU accelerated inference using NVIDIA GPUs
- Compatible with NVIDIA NIM containers
- Easy deployment through Unraid
- Supports modern LLM inference workloads

---
### Nexus Orchestrator

A self-hosted LLM orchestration layer that intelligently routes each prompt to the best local or cloud model — privacy-first, Ollama-native, no cloud defaults.

📄 Documentation:  
➡️ [Nexus Orchestrator Guide](docs/Nexus_UNRAID_GUIDE.md)

Features:

- Intelligent intent routing — classifies prompts (CODING, REASONING, CREATIVE, VISION, etc.) and dispatches to the right model automatically
- Hybrid local + cloud orchestration — per-category Local/Cloud provider toggle
- Vision and document support — attach images and files directly in chat
- Projects — organize conversations into named sidebar folders
- Privacy-first — all provider URLs and models start empty, 100% local operation supported

---

### Serena-MCP

A self-hosted MCP (Model Context Protocol) server that gives AI coding agents IDE-level code intelligence using language servers (LSP). Works with Claude Code, Claude Desktop, and any MCP-compatible client.

📄 Documentation:  
➡️ [Serena-MCP Guide](docs/SERENA-MCP.md)

Features:

- Semantic code navigation — find definitions, references, and symbols across your codebase
- Language server integration — supports Python, TypeScript, Go, Java, C/C++, PHP, and more
- MCP-compatible — works with Claude, Claude Code, and any MCP-enabled client
- Web dashboard — monitor and manage Serena via browser
- Streamable-HTTP transport — modern MCP transport protocol

---

### llama-swap

A lightweight proxy for `llama.cpp` that automatically swaps models in and out, making it easier to run multiple GGUF models on Unraid without keeping them all loaded at once.

📄 Documentation:  
➡️ [llama-swap Guide](docs/LLAMA-SWAP.md)

Features:

- Hot-swapping for `llama.cpp` / `llama-server`
- Helps manage multiple models
- Unraid-friendly config and model path setup
- Useful for local multi-model workflows

---

## Requirements

Some templates may require:

- **Unraid 6.12+**
- **NVIDIA GPU**
- **NVIDIA Driver Plugin for Unraid**
- Docker enabled on Unraid

---

## Installation

1. Install **Community Applications** in Unraid.
2. Add this repository as a template source or import the template manually.
3. Deploy the container from the Unraid Apps tab.

---

## Documentation

Each template has its own detailed documentation inside the `docs` folder.

---

## Contributing

Contributions are welcome.

---

## License 
See the [LICENSE](LICENSE) file for details.
This repository is provided as-is for the Unraid community.
