# JIT loading from dir

Example dir with models in .hf format:

```txt
/opt/vllm-models/
 ├── mistral-7b-instruct
 ├── llama2-7b-chat
 └── qwen-1.5b
```

Create `/opt/vllm-models/models.json` within the container and add a model list:

```json
[
  { "model": "mistral", "path": "/models/mistral-7b-instruct" },
  { "model": "llama2",  "path": "/models/llama2-7b-chat" },
  { "model": "qwen",    "path": "/models/qwen-1.5b" }
]
```

`"model"` is the name clients use to request a model to load. It can be anything.

Start the container / translate to dockerfile CMD:

```bash
docker run -d \
  --name vllm \
  --gpus all \
  -p 1234:1234 \
  -v /opt/vllm-models:/models \
  my-vllm:latest \
  python -m vllm.entrypoints.api_server \
    --host 0.0.0.0 \
    --port 1234 \
    --model-list /models/models.json \
    --api-key supersecret123 \
    --download-dir /models \
    --cpu-offload \
    --swap-space 16 \
    --gpu-memory-utilization 0.90

```

Load models using the openai-compatible REST API:

```bash
curl -X POST http://localhost:1234/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer supersecret123" \
  -d '{
    "model": "mistral",
    "messages": [
      {"role": "user", "content": "Explain containers in one paragraph"}
    ],
    "max_tokens": 128
  }'
```