The container is set-up with CUDA runtime 12.x.x, which matches with the loaded PyTorch version.

Build the container:

```dockerfile
FROM nvidia/cuda:12.3.2-cudnn-runtime-ubuntu22.04

# Basic build deps
RUN apt update && apt install -y \
    python3 python3-pip git && \
    rm -rf /var/lib/apt/lists/*

# Prevent pip warnings
RUN ln -s /usr/bin/python3 /usr/bin/python

# Install vLLM
RUN pip install --upgrade pip && \
    pip install vllm

# huggingface cli for private models
RUN pip install "huggingface_hub[cli]"

EXPOSE 8000

# Launch REST API server with a locked-in model (GPT OSS 20B)
CMD ["python", "-m", "vllm.entrypoints.api_server", "--host", "0.0.0.0", "--model", "openai/gpt-oss-20b"]
```

optional, but recommended for our use-case:

```dockerfile
# Launch REST API server with an initial model loaded (Mistral 7B Instruct) and a cache for 4 more models, which can be JIT-loaded.
CMD ["python", "-m", "vllm.entrypoints.api_server", "--host", "0.0.0.0", "--model", "mistralai/Mistral-7B-Instruct", "--model-cache-size 5"]
```