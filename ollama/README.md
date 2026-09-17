# [ollama](https://ollama.com/) [GITHUB](https://github.com/ollama/ollama)

## Download

```bash
# Windows powershell
irm https://ollama.com/install.ps1 | iex

# Linux / macOS 14 Sonoma or later
curl -fsSL https://ollama.com/install.sh | sh

# Packages
aria2c https://ollama.com/download/OllamaSetup.exe
aria2c https://ollama.com/download/Ollama-darwin.zip
aria2c https://ollama.com/download/ollama-linux-amd64.tar.zst

# Redirecting to
wget https://github.com/ollama/ollama/releases/latest/download/install.ps1
wget https://github.com/ollama/ollama/releases/latest/download/install.sh
aria2c https://github.com/ollama/ollama/releases/latest/download/OllamaSetup.exe
aria2c https://github.com/ollama/ollama/releases/latest/download/Ollama-darwin.zip
aria2c https://github.com/ollama/ollama/releases/latest/download/ollama-linux-amd64.tar.zst
```

## Run

```bash
ollama
ollama -v
```

## Use a coding agent (`Claude Code` / `Codex CLI` / `OpenCode`)

```bash
ollama launch claude
ollama launch codex --model qwen3.5
ollama launch opencode
```

## Build an application

### Create an API key

```bash
# Windows powershell
$env:OLLAMA_API_KEY = "your_api_key"

# Linux / macOS
export OLLAMA_API_KEY="your_api_key"
```

### Send a request

- Cloud models

```bash
curl https://ollama.com/api/tags
```

- Ollama API

```bash
curl https://ollama.com/api/chat \
  -H "Authorization: Bearer $OLLAMA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemma4:31b",
    "messages": [
      {
        "role": "user",
        "content": "Say hello in one sentence."
      }
    ],
    "stream": false
  }'
```

- OpenAI Chat Completions

```bash
curl https://ollama.com/v1/chat/completions \
  -H "Authorization: Bearer $OLLAMA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemma4:31b",
    "messages": [
      {
        "role": "user",
        "content": "Say hello in one sentence."
      }
    ]
  }'
```

- OpenAI Responses

```bash
curl https://ollama.com/v1/responses \
  -H "Authorization: Bearer $OLLAMA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemma4:31b",
    "input": "Say hello in one sentence."
  }'
```

- Anthropic Messages

```bash
curl https://ollama.com/v1/messages \
  -H "Authorization: Bearer $OLLAMA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "anthropic-version: 2023-06-01" \
  -d '{
    "model": "gemma4:31b",
    "max_tokens": 1024,
    "messages": [
      {
        "role": "user",
        "content": "Say hello in one sentence."
      }
    ]
  }'
```

## Run a model locally

```bash
ollama pull gemma4:e2b
ollama run gemma4:e2b # pull & run
ollama stop gemma4:e2b
ollama rm gemma4:e2b
ollama list # ollama ls
ollama ps
ollama serve # default port 11434 key ollama
```

- local api

```bash
curl http://localhost:11434/api/tags
curl http://localhost:11434/api/chat -H "" -d '{...}'
curl http://localhost:11434/v1/chat/completions -H "" -d '{...}'
curl http://localhost:11434/v1/responses -H "" -d '{...}'
curl http://localhost:11434/v1/messages -H "" -d '{...}'
```

## Resolve CORS

- add Environment in `ollama.service` file
  - `OLLAMA_HOST=0.0.0.0:11434`
  - `OLLAMA_ORIGINS=*`

## Create a customized model

```bash
cat > Modelfile <<EOF
FROM gemma4
SYSTEM """You are a happy cat."""
EOF

ollama create custom_model_name -f Modelfile
ollama show --modelfile custom_model_name
ollama run custom_model_name
```

## Streaming

- Python

```py
from ollama import chat

stream = chat(
  model='qwen3',
  messages=[{'role': 'user', 'content': 'What is 17 × 23?'}],
  stream=True,
)

in_thinking = False
content = ''
thinking = ''

for chunk in stream:
  if chunk.message.thinking:
    if not in_thinking:
      in_thinking = True
      print('Thinking:\n', end='', flush=True)
    print(chunk.message.thinking, end='', flush=True)
    # accumulate the partial thinking 
    thinking += chunk.message.thinking
  elif chunk.message.content:
    if in_thinking:
      in_thinking = False
      print('\n\nAnswer:\n', end='', flush=True)
    print(chunk.message.content, end='', flush=True)
    # accumulate the partial content
    content += chunk.message.content

  # append the accumulated fields to the messages for the next request
  new_messages = [{ role: 'assistant', thinking: thinking, content: content }]
```

- JavaScript

```js
import ollama from 'ollama'

async function main() {
  const stream = await ollama.chat({
    model: 'qwen3',
    messages: [{ role: 'user', content: 'What is 17 × 23?' }],
    stream: true,
  })

  let inThinking = false
  let content = ''
  let thinking = ''

  for await (const chunk of stream) {
    if (chunk.message.thinking) {
      if (!inThinking) {
        inThinking = true
        process.stdout.write('Thinking:\n')
      }
      process.stdout.write(chunk.message.thinking)
      // accumulate the partial thinking
      thinking += chunk.message.thinking
    } else if (chunk.message.content) {
      if (inThinking) {
        inThinking = false
        process.stdout.write('\n\nAnswer:\n')
      }
      process.stdout.write(chunk.message.content)
      // accumulate the partial content
      content += chunk.message.content
    }
  }

  // append the accumulated fields to the messages for the next request
  new_messages = [{ role: 'assistant', thinking: thinking, content: content }]
}

main().catch(console.error)
```

## Tool calling

- cURL

```bash
curl -s http://localhost:11434/api/chat -H "Content-Type: application/json" -d '{
  "model": "qwen3",
  "messages": [{"role": "user", "content": "What is the temperature in New York?"}],
  "stream": false,
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_temperature",
        "description": "Get the current temperature for a city",
        "parameters": {
          "type": "object",
          "required": ["city"],
          "properties": {
            "city": {"type": "string", "description": "The name of the city"}
          }
        }
      }
    }
  ]
}'

curl -s http://localhost:11434/api/chat -H "Content-Type: application/json" -d '{
  "model": "qwen3",
  "messages": [
    {"role": "user", "content": "What is the temperature in New York?"},
    {
      "role": "assistant",
      "tool_calls": [
        {
          "type": "function",
          "function": {
            "index": 0,
            "name": "get_temperature",
            "arguments": {"city": "New York"}
          }
        }
      ]
    },
    {"role": "tool", "tool_name": "get_temperature", "content": "22°C"}
  ],
  "stream": false
}'
```

- Python

```py
from ollama import chat

def get_temperature(city: str) -> str:
  """Get the current temperature for a city
  
  Args:
    city: The name of the city

  Returns:
    The current temperature for the city
  """
  temperatures = {
    "New York": "22°C",
    "London": "15°C",
    "Tokyo": "18°C",
  }
  return temperatures.get(city, "Unknown")

messages = [{"role": "user", "content": "What is the temperature in New York?"}]

# pass functions directly as tools in the tools list or as a JSON schema
response = chat(model="qwen3", messages=messages, tools=[get_temperature], think=True)

messages.append(response.message)
if response.message.tool_calls:
  # only recommended for models which only return a single tool call
  call = response.message.tool_calls[0]
  result = get_temperature(**call.function.arguments)
  # add the tool result to the messages
  messages.append({"role": "tool", "tool_name": call.function.name, "content": str(result)})

  final_response = chat(model="qwen3", messages=messages, tools=[get_temperature], think=True)
  print(final_response.message.content)
```

- JavaScript

```ts
import ollama from 'ollama'

function getTemperature(city: string): string {
  const temperatures: Record<string, string> = {
    'New York': '22°C',
    'London': '15°C',
    'Tokyo': '18°C',
  }
  return temperatures[city] ?? 'Unknown'
}

const tools = [
  {
    type: 'function',
    function: {
      name: 'get_temperature',
      description: 'Get the current temperature for a city',
      parameters: {
        type: 'object',
        required: ['city'],
        properties: {
          city: { type: 'string', description: 'The name of the city' },
        },
      },
    },
  },
]

const messages = [{ role: 'user', content: "What is the temperature in New York?" }]

const response = await ollama.chat({model: 'qwen3', messages, tools, think: true})

messages.push(response.message)
if (response.message.tool_calls?.length) {
  // only recommended for models which only return a single tool call
  const call = response.message.tool_calls[0]
  const args = call.function.arguments as { city: string }
  const result = getTemperature(args.city)
  // add the tool result to the messages
  messages.push({ role: 'tool', tool_name: call.function.name, content: result })

  // generate the final response
  const finalResponse = await ollama.chat({ model: 'qwen3', messages, tools, think: true })
  console.log(finalResponse.message.content)
}
```

## Structured Outputs

- cURL

```bash
curl -X POST http://localhost:11434/api/chat -H "Content-Type: application/json" -d '{
  "model": "gpt-oss",
  "messages": [{"role": "user", "content": "Tell me about Canada."}],
  "stream": false,
  "format": {
    "type": "object",
    "properties": {
      "name": {"type": "string"},
      "capital": {"type": "string"},
      "languages": {
        "type": "array",
        "items": {"type": "string"}
      }
    },
    "required": ["name", "capital", "languages"]
  }
}'
```

- Python (`Pydantic`)

```py
from ollama import chat
from pydantic import BaseModel

class Country(BaseModel):
  name: str
  capital: str
  languages: list[str]

response = chat(
  model='gpt-oss',
  messages=[{'role': 'user', 'content': 'Tell me about Canada.'}],
  format=Country.model_json_schema(),
)

country = Country.model_validate_json(response.message.content)
print(country)
```

- JavaScript (`Zod`)

```js
import ollama from 'ollama'
import * as z from 'zod'

const Country = z.object({
  name: z.string(),
  capital: z.string(),
  languages: z.array(z.string()),
})

const response = await ollama.chat({
  model: 'gpt-oss',
  messages: [{ role: 'user', content: 'Tell me about Canada.' }],
  format: z.toJSONSchema(Country),
})

const country = Country.parse(JSON.parse(response.message.content))
console.log(country)
```

## Vision

- cURL

```bash
IMG=$(base64 < test.jpg | tr -d '\n')

curl -X POST http://localhost:11434/api/chat \
-H "Content-Type: application/json" \
-d '{
    "model": "gemma4",
    "messages": [{
    "role": "user",
    "content": "What is in this image?",
    "images": ["'"$IMG"'"]
    }],
    "stream": false
}'
```

- Python

```py
from ollama import chat

path = input('Please enter the path to the image: ')

response = chat(
  model='gemma4',
  messages=[
    {
      'role': 'user',
      'content': 'What is in this image? Be concise.',
      'images': [path],
    }
  ],
)

print(response.message.content)
```

- JavaScript

```js
import ollama from 'ollama'

const imagePath = '/absolute/path/to/image.jpg'
const response = await ollama.chat({
  model: 'gemma4',
  messages: [
    { role: 'user', content: 'What is in this image?', images: [imagePath] }
  ],
  stream: false,
})

console.log(response.message.content)
```

## Embeddings

Generate text embeddings for semantic search, retrieval, and RAG.

- CLI

```bash
ollama run embeddinggemma "Hello world"
echo "Hello world" | ollama run embeddinggemma
```

- cURL

```bash
curl -X POST http://localhost:11434/api/embed \
  -H "Content-Type: application/json" \
  -d '{
    "model": "embeddinggemma",
    "input": ["First sentence", "Second sentence", "Third sentence"]
  }'
```

- Python

```py
import ollama

batch = ollama.embed(
  model='embeddinggemma',
  input=['First sentence', 'Second sentence', 'Third sentence']
)
print(len(batch['embeddings']))
```

- JavaScript

```js
import ollama from 'ollama'

const batch = await ollama.embed({
  model: 'embeddinggemma',
  input: ['First sentence', 'Second sentence', 'Third sentence'],
})
console.log(batch.embeddings.length)
```

## Web search

- `POST https://ollama.com/api/web_search`
  - request {query, max_results=5}
  - response {results[{title, url, content}]}

```py
import ollama
response = ollama.web_search("What is Ollama?")
print(response)
```

```js
import { Ollama } from "ollama";

const client = new Ollama();
const results = await client.webSearch("what is ollama?");
console.log(JSON.stringify(results, null, 2));
```

- `POST https://ollama.com/api/web_fetch`
  - request {url}
  - response {title, content, links[]}

```py
from ollama import web_fetch

result = web_fetch('https://ollama.com')
print(result)
```

```js
import { Ollama } from "ollama";

const client = new Ollama();
const fetchResult = await client.webFetch("https://ollama.com");
console.log(JSON.stringify(fetchResult, null, 2));
```
