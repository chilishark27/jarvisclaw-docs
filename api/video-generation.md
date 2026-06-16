# Video Generation

`POST /v1/videos/generations`

Generate videos from text prompts. Video generation is asynchronous — submit a job and poll for the result.

## Submit a Job

### Request

```json
{
  "model": "kling-v2",
  "prompt": "A cat walking gracefully on the moon's surface",
  "duration": 5
}
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | Yes | Video model ID |
| `prompt` | string | Yes | Text description of the video to generate |
| `duration` | integer | No | Video duration in seconds. Model-dependent |
| `size` | string | No | Video resolution (e.g., `1080x1920`) |
| `image` | string | No | Reference image URL for image-to-video |

### Response

```json
{
  "id": "video-job-abc123",
  "status": "processing",
  "created": 1700000000
}
```

## Poll for Result

`GET /v1/videos/generations/{id}`

### Response (completed)

```json
{
  "id": "video-job-abc123",
  "status": "completed",
  "url": "https://...",
  "created": 1700000000,
  "duration": 5
}
```

### Status Values

| Status | Meaning |
|--------|---------|
| `processing` | Job is in queue or actively generating |
| `completed` | Video is ready at the `url` field |
| `failed` | Generation failed (see `error` field) |

## Examples

::: code-group

```python [Python]
import time
from openai import OpenAI

client = OpenAI(
    base_url="https://api.jarvisclaw.ai/v1",
    api_key="sk-your-api-key",
)

# Submit job
response = client.post(
    "/videos/generations",
    body={
        "model": "kling-v2",
        "prompt": "A timelapse of a flower blooming",
    },
    cast_to=object,
)
job_id = response["id"]

# Poll until complete
while True:
    result = client.get(f"/videos/generations/{job_id}", cast_to=object)
    if result["status"] == "completed":
        print(f"Video URL: {result['url']}")
        break
    elif result["status"] == "failed":
        print(f"Error: {result['error']}")
        break
    time.sleep(5)
```

```bash [cURL]
# Submit
curl -X POST https://api.jarvisclaw.ai/v1/videos/generations \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"model": "kling-v2", "prompt": "A timelapse of a flower blooming"}'

# Poll (replace JOB_ID with the returned id)
curl https://api.jarvisclaw.ai/v1/videos/generations/JOB_ID \
  -H "Authorization: Bearer sk-your-api-key"
```

:::

## Notes

- Video generation takes 30 seconds to several minutes depending on the model and duration.
- Video URLs are temporary. Download the result promptly.
- Poll every 5-10 seconds to check status.
