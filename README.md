# `llama-hip` Container
This repository contains a `Containerfile.llama-hip` for building a containerized environment to run `llama.cpp` with AMD HIP support. The container is specifically configured for the `gfx1100` GPU architecture, which includes hardware like the AMD Radeon 780M Graphics.

---

### Prerequisites
* A system with an AMD GPU supported by ROCm.
* Podman installed.
* The `rocminfo` output confirms your GPU architecture is `gfx1100`.

---

### Build Instructions
To build the container image, use the `podman build` command. The `--no-cache` flag ensures a fresh build, and `--security-opt seccomp=unconfined` is required for ROCm compatibility.

```bash
podman build --no-cache --security-opt seccomp=unconfined -t llama-hip -f Containerfile.llama-hip .
```

### Run Instructions
After building the image, you can run the container. The command below includes all the necessary flags to expose your GPU and set the correct environment variables for llama.cpp to use HIPBLAS.

```bash
podman run -it --rm --name llama-service \
  --device /dev/kfd --device /dev/dri/renderD128 \
  --security-opt seccomp=unconfined \
  --group-add video \
  -p 8080:8080 \
  -v $HOME/.llama.cpp:/models:Z \
  -e GGML_HIPBLAS=1 \
  -e HSA_OVERRIDE_GFX_VERSION=11.0.0 \
  llama-hip \
  /llama.cpp/build/bin/llama-server -m /models/llama2-7b-chat.gguf --n-gpu-layers 32 --ctx-size 2048 --host 0.0.0.0
```

Note: Replace $HOME/.llama.cpp with the local path to your model files and llama2-7b-chat.gguf with your model's filename.

### Container Configuration
The Containerfile uses archlinux:base-devel as the base image and installs the following dependencies:

- git
- cmake
- python and python-pip
- rocm-hip-sdk, rocblas, hipblas, openmp, hip-runtime-amd

The llama.cpp project is cloned and built with specific cmake flags to enable HIP support:

-DGGML_HIP=ON
-DAMDGPU_TARGETS=gfx1100

The container's entry point is set to run the llama-server application, configured to use GPU layers and listen on port 8080. The HSA_OVERRIDE_GFX_VERSION=11.0.0 environment variable is also set to ensure compatibility.


You can use systemd, the standard service manager for Linux, to automatically start your Podman container on boot. Podman provides a convenient command to generate a systemd unit file directly from your run command.


Here's how to set up a systemd service for your llama-service container:

1. Generate the systemd Unit File
Instead of running the container, you will use your podman run command to generate a systemd service file. The --new flag ensures that the service creates a fresh container each time it starts, which is more reliable for a boot-time service.

podman generate systemd \
  --new \
  --files \
  --no-header \
  --container-prefix=llama-service \
  --device /dev/kfd --device /dev/dri/renderD128 \
  --security-opt seccomp=unconfined \
  --group-add video \
  -p 8080:8080 \
  -v $HOME/.llama.cpp:/models:Z \
  -e GGML_HIPBLAS=1 \
  -e HSA_OVERRIDE_GFX_VERSION=11.0.0 \
  llama-hip \
  /llama.cpp/build/bin/llama-server -m /models/llama2-7b-chat.gguf --n-gpu-layers 32 --ctx-size 2048 --host 0.0.0.0 > ~/.config/systemd/user/llama-service.service
  
  This command generates a systemd service file named llama-service.service and saves it to the standard user-level service directory.

2. Enable and Start the Service

Once the file is created, you can enable and start the service using systemctl.

Reload the systemd daemon to recognize the new service file:
Enable the service so it starts automatically at boot:

Bash

systemctl --user enable llama-service.service
Start the service immediately to test it:

Bash

systemctl --user start llama-service.service
You can check the status of your service with the command systemctl --user status llama-service.service. The --user flag is important because you are running a user-level service.
systemctl --user daemon-reload


