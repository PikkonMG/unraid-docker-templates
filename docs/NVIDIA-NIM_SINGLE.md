# NVIDIA NIM on Unraid

Run NVIDIA NIM inference microservices locally on Unraid using Docker. This guide covers setup, common errors, and connecting OpenAI-compatible clients.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Model Selection](#model-selection)
- [NGC Registry Login](#ngc-registry-login)
- [Docker Template Setup](#docker-template-setup)
- [Environment Variables](#environment-variables)
- [First Run](#first-run)
- [Connecting Clients](#connecting-clients)
- [Switching Models](#switching-models)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

- Unraid 6.12 or later
- NVIDIA GPU (Turing architecture or newer — GTX 16xx, RTX 20xx+)
- NVIDIA drivers installed in Unraid (Community Applications → NerdTools or GPU Statistics plugin)
- Free NGC account at [build.nvidia.com](https://build.nvidia.com)
- NGC API key generated at your NGC account dashboard

---

> **Tested on:** RTX 3060 12 GB · Unraid 6.12+ · NIM 1.10.1

## Model Selection

NIM uses pre-optimized engine profiles. Consumer GPUs require smaller models and reduced context windows. Below are Examples.

| Model | VRAM Required | Fits 12 GB? |
|-------|--------------|-------------|
| `meta/llama-3.2-3b-instruct` | ~6 GB | ✅ Recommended |
| `microsoft/phi-3-mini-4k-instruct` | ~8 GB | ✅ Yes |
| `nvidia/Llama-3.1-Nemotron-Nano-4B-v1.1` | ~10 GB | ✅ Yes |
| `mistralai/mistral-7b-instruct-v0.3` | ~14 GB fp16 | ❌ OOM |
| `meta/llama-3.1-8b-instruct` | ~22 GB bf16 | ❌ OOM |
| `meta/llama-3.1-70b-instruct` | ~80 GB | ❌ Multi-GPU only |

> For 7B+ models on a 12 GB consumer GPU, consider [Ollama](https://ollama.com) instead — it uses quantized weights and fits comfortably.

---

## NGC Registry Login

> ⚠️ **This must be done before Unraid can pull NIM images.** NIM images are hosted on NVIDIA's private registry (`nvcr.io`), not Docker Hub.

### One-time login via Unraid terminal

```bash
docker login nvcr.io
# Username: $oauthtoken       ← type this literally
# Password: YOUR_NGC_API_KEY
```

### Persist login across reboots

Add to `/boot/config/go` (runs at every boot):

```bash
docker login nvcr.io -u '$oauthtoken' -p 'YOUR_NGC_API_KEY'
```

> **Note:** `docker login` (image pull auth) and the `NGC_API_KEY` environment variable (runtime model weight download auth) are two **separate** authentications. Both are required.

---

## Docker Template Setup

In the Unraid Docker GUI, click **Add Container** and fill in the following fields.

### Basic Settings

| Field | Value |
|-------|-------|
| **Name** | `nvidia-nim` |
| **Repository** | `nvcr.io/nim/meta/llama-3.2-3b-instruct:latest` |
| **Network Type** | `bridge` |
| **Extra Parameters** | `--gpus all --shm-size=16gb --ulimit memlock=-1 --ulimit stack=67108864` |

### Port Mapping

| Container Port | Host Port | Protocol |
|---------------|-----------|----------|
| `8000` | `8000` | `TCP` |

### Volume Mapping

| Container Path | Host Path | Access Mode |
|---------------|-----------|-------------|
| `/opt/nim/.cache` | `/mnt/user/appdata/nvidia-nim/cache` | Read/Write |

**Before starting the container**, create the cache directory with correct permissions:

```bash
mkdir -p /mnt/user/appdata/nvidia-nim/cache
chown -R 1000:1000 /mnt/user/appdata/nvidia-nim/cache
chmod 775 /mnt/user/appdata/nvidia-nim/cache
```

---

## Environment Variables

| Variable | Value | Notes |
|----------|-------|-------|
| `NGC_API_KEY` | `your_ngc_api_key` | Required. Used at runtime to download model weights. |
| `NIM_MODEL_NAME` | `meta/llama-3.2-3b-instruct` | Must match the image tag. |
| `NIM_MAX_MODEL_LEN` | `16384` | **Required for consumer GPUs.** See |
| `NIM_CACHE_PATH` | `/opt/nim/.cache` | Points to the mounted cache volume. |
| `CUDA_VISIBLE_DEVICES` | `0` | Use `0` for single GPU. See |
| `PYTORCH_CUDA_ALLOC_CONF` | `expandable_segments:True` | Reduces memory fragmentation. |
| `NIM_LOG_LEVEL` | `INFO` | Set to `DEBUG` for verbose output. |

---

## First Run

On first start, NIM downloads model weights to the cache directory (~6 GB for the 3B model). This can take several minutes depending on your connection.

Watch the logs in the Unraid Docker UI or via terminal:

```bash
docker logs -f nvidia-nim
```

A successful startup looks like:

```
INFO:     Uvicorn running on http://0.0.0.0:8000
```

You can also verify the API is running:

```bash
curl http://localhost:8000/v1/models
```

---

## Connecting Clients

NIM exposes an OpenAI-compatible API. Use these settings in any compatible client:

| Setting | Value |
|---------|-------|
| **Docs** | `http://[unraid-ip]:8000/docs` |
| **Base URL** | `http://[unraid-ip]:8000/v1` |
| **API Key** | Any non-empty string (e.g. `nim`) — not validated locally |
| **Model** | `meta/llama-3.2-3b-instruct` |

### Compatible clients

- [AnythingLLM](https://github.com/Mintplex-Labs/anything-llm)
- [Open WebUI](https://github.com/open-webui/open-webui)
- [LangChain](https://python.langchain.com/)
- [LlamaIndex](https://www.llamaindex.ai/)
- [Cursor](https://www.cursor.com/) (custom OpenAI base URL)
- Any app with a configurable OpenAI-compatible endpoint

### Quick test

```bash
curl http://[unraid-ip]:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta/llama-3.2-3b-instruct",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

---

## Switching Models

NIM images for the template are currently model-specific — there is no in-app model browser. To switch:

1. Stop the existing container
2. Update the **Repository** field to the new model image (e.g. `nvcr.io/nim/microsoft/phi-3-mini-4k-instruct:latest`)
3. Update `NIM_MODEL_NAME` to match (e.g. `microsoft/phi-3-mini-4k-instruct`)
4. Start the container

To run **multiple models simultaneously**, create separate containers on different host ports (e.g. 8000, 8001). They can share the same cache folder — weights are not duplicated if the same model is used.

---

## Troubleshooting

CUDA_VISIBLE_DEVICES must be numeric

**Error:**
```
If you get following "ValueError: invalid literal for int() with base 10: 'all'" it's probably becuase you changed value to all!
```
**Fix:** Set `CUDA_VISIBLE_DEVICES=0` (not `all`). The `--gpus all` flag in Extra Parameters handles Docker-level GPU exposure separately.

---

Cache directory permission denied

**Error:**
```
The container will launch after creation you will probably get the following "PermissionError: [Errno 13] Permission denied: '/opt/nim/.cache/local_cache'"
```
**Fix:**
```bash
mkdir -p /mnt/user/appdata/nvidia-nim/cache
chmod 775 /mnt/user/appdata/nvidia-nim/cache
```

---

KV cache size error

**Error:**
```
If you get something like "ValueError: The model's max seq len (131072) is larger than the maximum number
of tokens that can be stored in KV cache (30320)".
```
**Fix:** Set `NIM_MAX_MODEL_LEN=16384`. If the error persists, try `8192`. Or 

Consumer GPUs cannot accommodate the full context window that data center profiles request. This variable caps it to a size that fits in available VRAM.

---

### General error reference

| Error | Cause | Fix |
|-------|-------|-----|
| `401 Unauthorized` / image pull fails | Not logged in to `nvcr.io` | Run `docker login nvcr.io` |
| `ValueError: invalid literal 'all'` | `CUDA_VISIBLE_DEVICES=all` | Change to `0` |
| `PermissionError` on `.cache` | Wrong directory permissions | `chmod 775` the cache path |
| `max seq len > KV cache` | Context window too large for GPU | Set `NIM_MAX_MODEL_LEN=16384` |
| `CUDA out of memory` | Model too large for GPU | Use a smaller model |
| `No compatible profiles` | GPU too old or driver too low | Requires Turing (RTX 20xx+) or newer |
| `WARNING: nvfp4 unsupported` | Consumer GPU lacks nvfp4 | Harmless — falls back to bf16 |

---

## XML Template

An Unraid Community Applications-compatible XML template is included in this repo as [`nvidia-nim.xml`](./nvidia-nim.xml). You can place it in `/boot/config/plugins/dockerMan/templates-user/` on your Unraid server to have it appear in the Docker template list.

---

## Resources

- [NVIDIA NIM Documentation](https://docs.nvidia.com/nim/)
- [NGC Model Catalog](https://catalog.ngc.nvidia.com/ai-foundation-models)
- [NIM API Reference](https://docs.nvidia.com/nim/large-language-models/latest/api-reference.html)
- [Unraid Docker Documentation](https://docs.unraid.net/unraid-os/manual/docker-management/)
