# Audio

Text-to-speech and speech-to-text endpoints.

## Text to Speech

`POST /v1/audio/speech`

Convert text to spoken audio.

### Request

```json
{
  "model": "tts-1",
  "input": "Hello, welcome to JarvisClaw!",
  "voice": "alloy"
}
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | Yes | TTS model ID (`tts-1`, `tts-1-hd`) |
| `input` | string | Yes | Text to convert to speech (max 4096 chars) |
| `voice` | string | Yes | Voice preset (e.g., `alloy`, `echo`, `fable`, `onyx`, `nova`, `shimmer`) |
| `response_format` | string | No | Audio format: `mp3` (default), `opus`, `aac`, `flac` |
| `speed` | float | No | Speed multiplier (0.25 to 4.0). Default: `1.0` |

### Response

Returns raw audio bytes with the appropriate `Content-Type` header.

### Example

::: code-group

```python [Python]
from openai import OpenAI

client = OpenAI(
    base_url="https://api.jarvisclaw.ai/v1",
    api_key="sk-your-api-key",
)

response = client.audio.speech.create(
    model="tts-1-hd",
    voice="nova",
    input="Hello, welcome to JarvisClaw!",
)

response.stream_to_file("output.mp3")
```

```bash [cURL]
curl https://api.jarvisclaw.ai/v1/audio/speech \
  -H "Authorization: Bearer sk-your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"model": "tts-1", "input": "Hello!", "voice": "alloy"}' \
  --output output.mp3
```

:::

## Speech to Text (Transcription)

`POST /v1/audio/transcriptions`

Transcribe audio files to text.

### Request (multipart/form-data)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file` | file | Yes | Audio file (mp3, mp4, mpeg, mpga, m4a, wav, webm) |
| `model` | string | Yes | STT model ID (`whisper-1`) |
| `language` | string | No | ISO-639-1 language code (improves accuracy) |
| `response_format` | string | No | `json` (default), `text`, `srt`, `verbose_json`, `vtt` |
| `temperature` | float | No | Sampling temperature (0-1) |

### Response

```json
{
  "text": "Hello, this is a transcription of the audio file."
}
```

### Example

::: code-group

```python [Python]
from openai import OpenAI

client = OpenAI(
    base_url="https://api.jarvisclaw.ai/v1",
    api_key="sk-your-api-key",
)

with open("recording.mp3", "rb") as audio_file:
    transcript = client.audio.transcriptions.create(
        model="whisper-1",
        file=audio_file,
    )
    print(transcript.text)
```

```bash [cURL]
curl https://api.jarvisclaw.ai/v1/audio/transcriptions \
  -H "Authorization: Bearer sk-your-api-key" \
  -F file="@recording.mp3" \
  -F model="whisper-1"
```

:::

## Notes

- Maximum audio file size: 25 MB
- Supported audio formats: mp3, mp4, mpeg, mpga, m4a, wav, webm
- For long audio, split into segments under 25 MB each
