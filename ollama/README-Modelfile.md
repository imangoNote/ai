# ollama - Modelfile

A Modelfile is the blueprint to create and share customized models using Ollama. The Modelfile is not case sensitive.

## Create a customized model

```bash
ollama create custom_model_name -f Modelfile
ollama show --modelfile custom_model_name
ollama run custom_model_name
```

## Format of the Modelfile

```text
# comment
INSTRUCTION arguments
```

| Instruction | Description |
| ----- | ----- |
| FROM (required) | Defines the base model to use. |
| PARAMETER | Sets the parameters for how Ollama will run the model. |
| TEMPLATE | The full prompt template to be sent to the model. |
| SYSTEM | Specifies the system message that will be set in the template. |
| LICENSE | Specifies the legal license. |
| MESSAGE | Specify message history. |
| REQUIRES | Specify the minimum version of Ollama required by the model. |

## Instructions

### FROM (Required)

```text
FROM <model name>:<tag>
```

- **Build from existing model:** `FROM llama3.2`
- **Build from a Safetensors model:** `FROM path_to/xxxx.safetensors`
- **Build from a GGUF model:** `FROM path_to/xxxx.gguf`

### PARAMETER

```text
PARAMETER <parameter> <parametervalue>
```

- valid parameter
  - `num_ctx 2048` Sets the size of the context window used to generate the next token.
  - `repeat_last_n 64` Sets how far back for the model to look back to prevent repetition.
  - `repeat_penalty 1.0` Sets how strongly to penalize repetitions.
  - `temperature 0.8` Increasing the temperature will make the model answer more creatively.
  - `seed 0` Sets a specific number will make the model generate the same text for the same prompt.
  - `stop "QUIT"` When this pattern is encountered the LLM will stop generating text and return.
  - `num_predict -1` Maximum number of tokens to predict when generating text.
  - `draft_num_predict 4` Maximum number of speculative draft tokens to predict per step when a draft model is available.
  - `top_k 40` Reduces the probability of generating nonsense.
  - `top_p 0.9` Works together with top-k.
  - `min_p 0.0` Alternative to the top-p, and aims to ensure a balance of quality and variety. 

### TEMPLATE

| Variable | Description |
| ----- | ----- |
| {{ .System }} | The system message used to specify custom behavior. |
| {{ .Prompt }} | The user prompt message. |
| {{ .Response }} | The response from the model. <br>When generating a response, text after this variable is omitted. |

```text
TEMPLATE """{{ if .System }}<|im_start|>system
{{ .System }}<|im_end|>
{{ end }}{{ if .Prompt }}<|im_start|>user
{{ .Prompt }}<|im_end|>
{{ end }}<|im_start|>assistant
"""
```

### SYSTEM

The `SYSTEM` instruction specifies the system message to be used in the template, if applicable.

```text
SYSTEM """<system message>"""
```

### LICENSE

The LICENSE instruction allows you to specify the legal license under which the model used with this Modelfile is shared or distributed.

```text
LICENSE """
<license text>
"""
```

### MESSAGE

The MESSAGE instruction allows you to specify a message history for the model to use when responding.

```text
MESSAGE <role> <message>
```

- valid roles: `system` / `user` / `assistant`

### REQUIRES

The REQUIRES instruction allows you to specify the minimum version of Ollama required by the model.

```text
REQUIRES <version>
```
