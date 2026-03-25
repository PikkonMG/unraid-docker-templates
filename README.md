# Unraid Docker Templates

A collection of Docker templates designed for **Unraid**.  
These templates make it easier to deploy advanced self-hosted tools with minimal configuration.

This repository focuses on providing **well-structured templates and documentation** so users can quickly deploy services through the Unraid GUI.

---

## Available Templates

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
