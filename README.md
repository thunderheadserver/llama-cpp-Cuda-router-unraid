# llama.cpp CUDA Router — Unraid template

A native Unraid Docker template for llama.cpp's CUDA server in multi-model Router Mode.

## Description

llama.cpp Docker container for NVIDIA GPUs. Configured in Router Mode to support multiple models with automatic model loading and unloading.

Includes configurable inference settings such as context size, GPU offloading, KV-cache precision, reasoning controls, Jinja templates, and parallel processing.

## Maintenance note

> Not actively and/or very poorly maintained because I don't know what the hell I'm doing.

## Installation considerations

- Uses `ghcr.io/ggml-org/llama.cpp:server-cuda` and needs the NVIDIA container runtime on Unraid.
- Models and presets are stored in a persistent host path (template default: `/mnt/user/appdata/llama_cpp/model`).
- Router presets: `/models/models.ini` (host: `.../model/models.ini`). **The template does not create this file**. Supply a valid `version = 1` INI with at least one `[Model-ID]` section and `model = /models/FILE.gguf`, or adapt the configuration for your model source.
- The template has been reviewed for XML structure but **has not been installation-tested as a new container**. The underlying working setup was tested on Thunderhead; review router worker inheritance of inference settings.
- The NVIDIA runtime and health check are supplied as Docker extra parameters. Port 8000 is used by the default health check, so coordinate any internal port changes.
- The default paths are examples and are specific to Thunderhead's layout. Customize for other systems.
- The 30-minute idle/sleep setting discussed during development is not included because its global/router-worker inheritance has not been confirmed on the actual deployed build.

## Example models.ini

```ini
version = 1

[Qwen3.5-9B-Q4KM-16K]
model = /models/qwen-3.5-9b-q4_k_m.gguf

[Gemma4-12B-QAT-Q4_0-16K]
model = /models/gemma-4-12b-it-qat-q4_0.gguf
```

Model files aren't included; obtain appropriately licensed GGUFs separately.

## Before public Community Applications submission

1. Upload **your own** icon to `icons/llama-cpp.png`.
2. Add an `<Icon>` to the Docker template pointing to the PNG's GitHub raw URL.
3. Add `<TemplateURL>` pointing to the template XML's GitHub raw URL.
4. Update `ca_profile.xml` with a repository `<Icon>` and `<WebPage>`.
5. Add an OSI-approved `LICENSE` file with your own author details; none is included in this draft.
6. Verify the container runs from a fresh install, and run Community Apps **Validate** and **Scan** before submission.

Submission info: https://ca.unraid.net/submit/help

## Local-only review

Copy `templates/llama-cpp-cuda-router.xml` into `/boot/config/plugins/dockerMan/templates-user/` with a distinct filename. Review in Docker → Add Container **without clicking Apply** to avoid colliding with a running instance.
