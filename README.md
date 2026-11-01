llama.cpp CUDA Router — Unraid
Unraid Docker template for llama.cpp on NVIDIA GPUs, configured for multi-model Router Mode. Models load on demand; the template exposes settings for GPU offloading, context size, KV-cache precision, Jinja chat templates, reasoning controls, and parallel slots.
Maintenance note: Not actively and/or very poorly maintained because I don't know what the hell I'm doing.

Requirements
- A supported NVIDIA GPU, working NVIDIA drivers, and the NVIDIA container runtime configured in Unraid.
- Disk space for GGUF models and the model download cache. Models are not bundled with this template.
The image is ghcr.io/ggml-org/llama.cpp:server-cuda.
Install on Unraid
1. Install this template from Community Applications when available. Before publication, you can import the XML from [`templates/llama-cpp-cuda-router.xml`](templates/llama-cpp-cuda-router.xml) as a custom Unraid Docker template.
2. In the Docker editor, review API / Web UI Port (default 8000) and Model Storage Directory (default /mnt/user/appdata/llama_cpp/model). Choose a host port that isn't already used by another container.
3. Leave Model Directory set to /models for automatic discovery of GGUF files in the storage folder. Leave Model Presets File (Optional) empty for a new installation.
4. Confirm NVIDIA GPU Devices (default all) and the other GPU settings match your hardware. Leave Maximum Loaded Models at 1 if you want only one model resident at a time.
5. Click Apply to create/start the container. When Unraid shows it as healthy, open its Web UI from the Docker tab (or visit http://UNRAID-IP:HOST-PORT).
6. Add a model using either method below, then select it in the Web UI or your OpenAI-compatible client.
Port note: The host-side API/Web UI port may be changed, but the internal llama.cpp listening port should remain 8000 unless you also update the container port mapping and the health-check command.
Add a model without presets
Method A — Built-in Web UI
Open the server's Web UI and use its model-management/download controls to obtain a supported GGUF from Hugging Face. Downloads use llama.cpp's cache. The template exposes a persistent Model Download Cache Directory mapping so downloads can survive container recreation. The available Web UI controls can vary with the llama.cpp image version.
Method B — Copy a GGUF to the model directory
Copy a .gguf file into the host folder configured as Model Storage Directory (for example, /mnt/user/appdata/llama_cpp/model/). The router discovers models from this directory through LLAMA_ARG_MODELS_DIR=/models. Restart the container if the new model does not appear in its list.

Use descriptive filenames, such as qwen-3.5-9b-q4_k_m.gguf, so model selections are recognizable in clients. Models installed this way do not need models.ini.

For models with multiple GGUF parts or separate multimodal projector files, follow llama.cpp's documented model-directory layout rather than treating every GGUF as an independent model.
Optional: Custom Model Names and Settings
By default, llama.cpp identifies models using their discovered filenames or model identifiers. These can be difficult to recognize, particularly when running multiple models with different sizes or quantizations.

Model presets (models.ini) allow you to assign descriptive, consistent names to individual models.

For example, instead of seeing:

model.gguf

An application connected to llama.cpp could display:

Qwen3.5-9B-Q4KM-16K
Why use model presets?
- Easier model selection: Identify a model's family, size, quantization, and intended context configuration directly from its name.
- Comparing and benchmarking models: Easily distinguish models and quantizations when evaluating response quality, generation speed, token throughput, memory usage, and other performance characteristics.
- Consistent API identifiers: Keep the same model name in connected applications even when the underlying GGUF filename changes.
- Model-specific settings: Assign different inference parameters to individual models when shared server settings aren't appropriate.
Example configuration
Create a file named models.ini inside your model storage directory:
version = 1

[Qwen3.5-9B-Q4KM-16K]
model = /models/qwen-3.5-9b-q4_k_m.gguf

[Gemma4-12B-QAT-Q4_0-16K]
model = /models/gemma-4-12b-it-qat-q4_0.gguf
The names in square brackets become the model identifiers exposed through llama.cpp's API.

To enable the presets, enter /models/models.ini in the Model Presets File (Optional) setting in Unraid, then apply the changes. Only use entries for model files that actually exist. If directory discovery and presets are both enabled, check for duplicate listings before relying on a model ID in another application.

Presets are not required for normal operation. If the automatically discovered model names are already clear enough, you can leave this setting empty and use directory-based discovery.
Settings at a glance
Section	Main controls
Server	Host API/Web UI port, bind address, internal listening port
Models	Model storage, directory discovery, optional preset file, maximum loaded models, autoload, persistent download cache
GPU	NVIDIA devices and capabilities, GPU offload layers, KV-cache key/value precision
Inference	Context size, parallel slots, Jinja templates, reasoning mode, reasoning format, reasoning budget


Model autoload means a model is loaded when selected for a request; it does not mean every available model is loaded at container startup. The default Maximum Loaded Models = 1 limits simultaneous resident models, not how many models can be listed.

Reasoning settings may behave differently across model families and llama.cpp versions. Validate them with the model you intend to use.
Verified testing and limitations
The template was installed and tested on an Unraid NVIDIA system using a separate host port (8001 mapped to container port 8000). Testing confirmed:
- Docker health status healthy and /health returning HTTP 200.
- The NVIDIA Docker runtime was selected.
- Qwen3.5 9B and Gemma 4 12B appeared with named preset IDs.
- Both models generated successful API responses.
- Switching to Gemma left Gemma loaded and Qwen unloaded with the loaded-model limit set to 1.
Not yet verified: a fresh install with no presets and no existing GGUF files; the new directory-only discovery configuration; every inference setting's inheritance by router worker processes; and the model-download cache path on all image versions. Don't treat those behaviors as verified solely by the tests above.
Troubleshooting
- No models listed: Confirm the GGUF files are in the host folder mapped to /models; check the Model Directory setting; restart and check the container logs.
- Container doesn't start or GPU isn't used: Verify the NVIDIA driver and container runtime are installed and functional, then check the Unraid Docker logs.
- Port already in use: Change the host API / Web UI Port to an available port; leave the internal port at 8000.
- Unhealthy container: Check /health on the container's internal port 8000 and review the Docker logs.
- Different model names than expected: Directory discovery uses model-derived names. Define aliases in an optional models.ini if you need stable custom IDs.
Links
- Upstream llama.cpp
- llama.cpp server documentation
- This Unraid template repository
This repository packages a Docker template for the upstream llama.cpp image. It does not contain or redistribute GGUF model weights.
