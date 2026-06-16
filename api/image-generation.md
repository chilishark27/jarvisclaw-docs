# Image Generation

`POST /v1/images/generations`

Generate images from text prompts.

## Request

```json
{
  "model": "dall-e-3",
  "prompt": "A cyberpunk cityscape at sunset, neon lights reflecting on wet streets",
  "n": 1,
  "size": "1024x1024",
  "quality": "hd"
}
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | Yes | Image model ID (e.g., `dall-e-3`, `stable-diffusion-xl`) |
| `prompt` | string | Yes | Text description of the image to generate |
| `n` | integer | No | Number of images to generate. Default: `1` |
| `size` | string | No | Image dimensions. Options vary by model (e.g., `1024x1024`, `1792x1024`) |
| `quality` | string | No | Quality level (`standard` or `hd`). Model-dependent |
| `response_format` | string | No | `url` (default) or `b64_json` |
| `style` | string | No | Style preset (`vivid` or `natural`). Model-dependent |

## Response

```json
{
  "created": 1700000000,
  "data": [
    {
      "url": "https://...",
      "revised_prompt": "A detailed cyberpunk cityscape..."
    }
  ]
}
```

## Examples

::: code-group

```python [Python]
from openai import OpenAI

client = OpenAI(
    base_url="https://api.jarvisclaw.ai/v1",
    api_key="sk-your-api-key",
)

response = client.images.generate(
    model="dall-e-3",
    prompt="A serene Japanese garden with cherry blossoms",
    size="1024x1024",
    quality="hd",
)

print(response.data[0].url)
```

```bash [cURL]
curl https://api.jarvisclaw.ai/v1/images/generations \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "dall-e-3",
    "prompt": "A serene Japanese garden with cherry blossoms",
    "size": "1024x1024",
    "quality": "hd"
  }'
```

:::

## Notes

- Image URLs are temporary and expire. Download images promptly or use `response_format: "b64_json"`.
- Available sizes and quality options depend on the specific model.
- Some models support `revised_prompt` in the response showing the expanded prompt used internally.
