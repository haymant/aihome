
# Ollama at Home

## Install Ollama

Follow instruction at [Ollama](https://ollama.com/download)


## Install models

For existing setup, can list models installed:
```bash
$ OLLAMA_HOST=0.0.0.0 ollama list
NAME              ID              SIZE      MODIFIED   
qwen2.5:latest    845dbda0ea48    4.7 GB    6 days ago    
deepseek-r1:7b    0a8c26691023    4.7 GB    6 days ago  
```

This command install deepseek r1 and start the service.

```bash
sudo systemctl stop ollama
OLLAMA_HOST=0.0.0.0 ollama run nomic-embed-text:latest
OLLAMA_HOST=0.0.0.0 ollama run deepseek-r1:7b
OLLAMA_HOST=0.0.0.0 ollama serve
```

Please docker container on Linux, we can reach ollama at http://192.168.1.99:11434/

Verify via CLI:

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "deepseek-r1:7b",
  "messages": [{ "role": "user", "content": "Solve: 25 * 25" }],
  "stream": false
}'
```