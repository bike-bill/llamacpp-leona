## Docker Image

ghcr.io/ggml-org/llama.cpp:server-rocm

## Usage

docker run -v ~/.models:/models ghcr.io/ggml-org/llama.cpp:full --all-in-one "/models/" 7B

## Local Usage

HSA_OVERRIDE_GFX_VERSION=11.0.0 ROCR_VISIBLE_DEVICES=0 llama-server \
  -m ~/.models/llama2-7b-chat.gguf \
  --n-gpu-layers 4 \
  --threads 14 \
  --ctx-size 2048 \
  --batch-size 1024 \
  --mlock \
  --rope-freq-base 20000 \
  --temp 0.8

## Monitoring

```watch -n 1 rocm-smi```
```rocminfo```


