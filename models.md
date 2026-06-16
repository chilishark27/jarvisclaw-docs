# Models

JarvisClaw aggregates models from many providers behind a single API. The model catalog is dynamic — models are added and updated regularly.

## Listing Available Models

Use the standard `/v1/models` endpoint to get the current list:

::: code-group

```python [Python]
from openai import OpenAI

client = OpenAI(
    base_url="https://api.jarvisclaw.ai/v1",
    api_key="sk-your-api-key",
)

models = client.models.list()
for model in models.data:
    print(model.id)
```

```bash [cURL]
curl https://api.jarvisclaw.ai/v1/models \
  -H "Authorization: Bearer sk-your-api-key"
```

:::

## Providers

Models are sourced from providers including:

- **OpenAI** — GPT-4o, GPT-4o-mini, o1, o3, DALL-E, Whisper, TTS
- **Anthropic** — Claude Opus, Sonnet, Haiku
- **Google** — Gemini Pro, Flash, Ultra
- **DeepSeek** — DeepSeek-V3, DeepSeek-R1
- **Meta** — Llama models
- **Mistral** — Mistral Large, Codestral
- **Stability AI** — Stable Diffusion, Stable Video
- **Kling** — Video generation
- **And more** — new providers added continuously

## Model Types

| Type | Endpoint | Examples |
|------|----------|----------|
| Chat/LLM | `/v1/chat/completions` | gpt-4o, claude-sonnet, gemini-pro |
| Image | `/v1/images/generations` | dall-e-3, stable-diffusion-xl |
| Video | `/v1/videos/generations` | kling-v2 |
| Audio TTS | `/v1/audio/speech` | tts-1, tts-1-hd |
| Audio STT | `/v1/audio/transcriptions` | whisper-1 |
| Embeddings | `/v1/embeddings` | text-embedding-3-small |

## Smart Router Aliases

Instead of picking a specific model, use routing aliases:

- `auto` — best quality/cost balance
- `free` — free-tier models only
- `eco` — cheapest paid option
- `premium` — highest capability

See [Smart Router](/smart-router) for details.
