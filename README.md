# LiteLLM Proxy Installation Guide

This guide will help you set up LiteLLM as a proxy between Ollama and OpenWebUI.

## Prerequisites

- Docker installed and running
- Ollama container running
- OpenWebUI container running (optional, for UI integration)

## Installation Steps

1. **Clone or create the litellm-proxy directory**
   ```bash
   mkdir -p ~/litellm-proxy
   cd ~/litellm-proxy
   ```

2. **Find your Ollama container's network**
   ```bash
   docker ps | grep ollama
   docker inspect ollama-container-name | grep -A 10 "Networks"
   ```

3. **Start the LiteLLM proxy**
   ```bash
   docker compose up -d
   ```

4. **Verify the installation**
   ```bash
   # Check if it's running
   docker ps | grep litellm-proxy

   # Verify the models are available
   curl http://localhost:8000/v1/models \
     -H "Authorization: Bearer sk-1234567890abcdef"
   ```

5. **Configure OpenWebUI (Optional)**

   Update your OpenWebUI container to use the LiteLLM proxy:
   - Change `OLLAMA_BASE_URL` to `http://litellm-proxy:8000` (if on same network) or `http://host.docker.internal:8000`
   - Add `OPENAI_API_KEY=sk-1234567890abcdef`

6. **Test the setup**
   ```bash
   curl http://localhost:8000/v1/chat/completions \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer sk-1234567890abcdef" \
     -d '{
       "model": "llama3.2:1b",
       "messages": [{"role": "user", "content": "Hello, how are you?"}]
     }'
   ```

## Customization Options

- **API Key**: Change `sk-1234567890abcdef` in both files to your preferred key
- **Port**: Change `8000:8000` in docker-compose.yml if you want to use a different port
- **Models**: Update the model names in config.yaml to match your available Ollama models
- **Database**: The setup includes a PostgreSQL 17 database container. You can customize the database credentials in the docker-compose.yml file.

# LiteLLM Proxy API Examples

This README contains sample curl commands for interacting with the LiteLLM proxy API. The proxy provides an OpenAI-compatible API for your Ollama models.

## Configuration

- **API Base URL**: http://localhost:8000
- **API Key**: sk-1234567890abcdef
- **Available Models**: 
  - llama3.2:1b
  - llama3.2:latest

## API Examples

### List Available Models

```bash
curl http://localhost:8000/v1/models \
  -H "Authorization: Bearer sk-1234567890abcdef"
```

### Chat Completions

Basic chat completion:

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-1234567890abcdef" \
  -d '{
    "model": "llama3.2:1b",
    "messages": [{"role": "user", "content": "Hello, how are you?"}]
  }'
```

With temperature control:

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-1234567890abcdef" \
  -d '{
    "model": "llama3.2:latest",
    "messages": [{"role": "user", "content": "Write a short poem about the ocean"}],
    "temperature": 0.7
  }'
```

With system message:

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-1234567890abcdef" \
  -d '{
    "model": "llama3.2:latest",
    "messages": [
      {"role": "system", "content": "You are a helpful assistant that speaks like a pirate."},
      {"role": "user", "content": "Tell me about the weather today"}
    ]
  }'
```

### Streaming Chat Completions

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-1234567890abcdef" \
  -d '{
    "model": "llama3.2:1b",
    "messages": [{"role": "user", "content": "Write a short story about a robot"}],
    "stream": true
  }'
```

### Controlling Generation Parameters

With max tokens:

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-1234567890abcdef" \
  -d '{
    "model": "llama3.2:latest",
    "messages": [{"role": "user", "content": "Explain quantum computing"}],
    "max_tokens": 100
  }'
```

With top_p sampling:

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-1234567890abcdef" \
  -d '{
    "model": "llama3.2:1b",
    "messages": [{"role": "user", "content": "Generate a creative story idea"}],
    "top_p": 0.8
  }'
```

### Multi-turn Conversations

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-1234567890abcdef" \
  -d '{
    "model": "llama3.2:latest",
    "messages": [
      {"role": "user", "content": "Hello, I want to learn about Mars"},
      {"role": "assistant", "content": "Mars is the fourth planet from the Sun and the second-smallest planet in the Solar System. It is often called the Red Planet due to its reddish appearance. What specific aspects of Mars would you like to know about?"},
      {"role": "user", "content": "Tell me about its moons"}
    ]
  }'
```

### Using Stop Sequences

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-1234567890abcdef" \
  -d '{
    "model": "llama3.2:1b",
    "messages": [{"role": "user", "content": "Write a numbered list of planets"}],
    "stop": ["5."]
  }'
```

## Using with Python

If you prefer using Python instead of curl, here's a simple example:

```python
import openai

client = openai.OpenAI(
    api_key="sk-1234567890abcdef",
    base_url="http://localhost:8000/v1"
)

response = client.chat.completions.create(
    model="llama3.2:latest",
    messages=[
        {"role": "user", "content": "Hello, how are you?"}
    ]
)

print(response.choices[0].message.content)
```

## Using with Node.js

```javascript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: 'sk-1234567890abcdef',
  baseURL: 'http://localhost:8000/v1',
});

async function main() {
  const completion = await openai.chat.completions.create({
    model: 'llama3.2:latest',
    messages: [
      { role: 'user', content: 'Hello, how are you?' }
    ],
  });

  console.log(completion.choices[0].message.content);
}

main();
```

## Troubleshooting

If you encounter issues:

1. Check that the LiteLLM proxy container is running:
   ```bash
   docker ps | grep litellm-proxy
   ```

2. Verify the models are available:
   ```bash
   curl http://localhost:8000/v1/models \
     -H "Authorization: Bearer sk-1234567890abcdef"
   ```

3. Check the logs for errors:
   ```bash
   docker logs litellm-proxy
   ```

4. Database issues:
   ```bash
   # If you see database-related errors, check your DATABASE_URL setting and make sure the PostgreSQL container is running
   docker logs litellm-postgres
   ```
