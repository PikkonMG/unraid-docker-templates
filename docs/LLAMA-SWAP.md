# llama-swap on Unraid

Run `llama-swap` on Unraid to expose a single OpenAI-compatible endpoint that can hot-swap between multiple `llama.cpp` models on demand.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Template Variants](#template-variants)
- [Docker Template Setup](#docker-template-setup)
- [Config File Setup](#config-file-setup)
- [First Run](#first-run)
- [Using llama-swap](#using-llama-swap)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

- Unraid 6.12 or later
- A folder for your GGUF models
- A `config.yaml` file in your llama-swap config directory
- A compatible backend command in your config, typically `llama-server`

Optional:

- NVIDIA GPU with the Unraid NVIDIA plugin for the `cuda` tag
- AMD GPU for the `rocm` tag
- Intel or other Vulkan-capable GPU for the `vulkan` tag

> **Included example:** [example-llama-swap-config.yaml](../examples/llama-swap/example-llama-swap-config.yaml)

---

## Template Variants

The template includes multiple repository tags depending on how you want to run inference:

| Tag | Use Case | Extra Parameters |
|-----|----------|------------------|
| `cuda` | NVIDIA GPU acceleration | `--gpus all` |
| `cpu` | CPU-only systems | Leave empty |
| `rocm` | AMD GPU acceleration | `--device=/dev/kfd --device=/dev/dri --group-add video --security-opt seccomp=unconfined` |
| `vulkan` | Intel or Vulkan-capable GPU acceleration | `--device=/dev/dri` |

---

## Docker Template Setup

In Unraid, install the template from Community Applications or import the XML manually.

### Basic Settings

| Field | Value |
|-------|-------|
| **Name** | `llama-swap` |
| **Repository** | `ghcr.io/mostlygeek/llama-swap:cuda` |
| **Network Type** | `bridge` |
| **Post Arguments** | `-config /config/config.yaml -watch-config` |

Change the repository tag if you want `cpu`, `rocm`, or `vulkan` instead of `cuda`.

### Port Mapping

| Container Port | Host Port | Protocol |
|---------------|-----------|----------|
| `8080` | `8080` | `TCP` |

### Volume Mapping

| Container Path | Host Path | Access Mode |
|---------------|-----------|-------------|
| `/models` | `/mnt/user/appdata/llama-swap/models` | Read Only |
| `/config` | `/mnt/user/appdata/llama-swap/config` | Read/Write |

### Optional Environment Variables

| Variable | Default | Notes |
|----------|---------|-------|
| `NVIDIA_VISIBLE_DEVICES` | `all` | Only relevant for NVIDIA setups |
| `NVIDIA_DRIVER_CAPABILITIES` | `all` | Use `compute,utility` or `all` depending on your setup |

---

## Config File Setup

Create this file:

```text
/mnt/user/appdata/llama-swap/config/config.yaml
```

Use the repository example as your starting point:

- [llama-swap example config](../examples/llama-swap/example-llama-swap-config.yaml)

When editing `config.yaml`:

- Reference model files as `/models/your-model.gguf`
- Make sure the backend command inside the config matches the binaries available in your chosen image
- Keep `-watch-config` enabled if you want config changes to reload automatically

---

## First Run

1. Place your `.gguf` files in `/mnt/user/appdata/llama-swap/models`
2. Create or update `/mnt/user/appdata/llama-swap/config/config.yaml`
3. Start the container from the Unraid Docker page
4. Open the web UI at `http://[unraid-ip]:8080/ui`

You can verify the API is reachable at:

```bash
curl http://[unraid-ip]:8080/v1/models
```

---

## Using llama-swap

`llama-swap` sits in front of `llama.cpp` backend and loads models on demand based on your config. This lets you keep one endpoint while switching among multiple models without manually restarting containers.


Screenshot:

- [llama-swap UI screenshot](screenshots/llama-swap-screenshot1.png)

---

## Troubleshooting

### Models do not load

Check that:

- Your GGUF files exist in the host models directory
- The config references them as `/models/...`
- The selected tag matches your hardware and runtime flags

### NVIDIA container fails to use the GPU

Check that:

- The Unraid NVIDIA plugin is installed
- You are using the `cuda` tag
- Extra Parameters contains `--gpus all`

### AMD or Intel GPU acceleration does not start

Check that:

- `rocm` uses `--device=/dev/kfd --device=/dev/dri --group-add video --security-opt seccomp=unconfined`
- `vulkan` uses `--device=/dev/dri`

### Config changes do not apply

Make sure the container is started with:

```text
-config /config/config.yaml -watch-config
```
