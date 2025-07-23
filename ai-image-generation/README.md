# AI Image Generation

This example demonstrates how to use Kong's AI Proxy Advanced plugin to create an AI gateway for image generation and audio transcription services using OpenAI's APIs.

## Overview

This setup showcases Kong Gateway as an AI proxy that can:
- Route image generation requests to OpenAI's DALL-E 3 model
- Handle audio transcription requests using OpenAI's Whisper model
- Provide load balancing, retries, and logging for AI requests
- Manage API keys and authentication centrally

## Prerequisites

**Software Requirements:**
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [curl](https://curl.se/) (for testing)

**Kong Requirements:**
- Kong Enterprise license (for AI Proxy Advanced plugin)
- OpenAI API key

## Setup

### 1. Configure Environment

Choose your deployment method:

**Option A: Kong Konnect (Recommended)**
```bash
# Copy and edit the Konnect environment file
cp konnect.env.example konnect.env
# Edit konnect.env with your Konnect credentials
```

**Option B: Kong Enterprise**
```bash
# Copy and edit the Enterprise environment file
cp ee.env.example ee.env
# Edit ee.env with your Kong Enterprise license
```

### 2. Configure OpenAI API Key

Edit the Kong configuration file to add your OpenAI API key:

```bash
# Edit kong-config/kong.yaml
# Replace <your-api-key> with your actual OpenAI API key
```

### 3. Start the Services

**For Konnect:**
```bash
docker compose up -d
```

**For Kong Enterprise:**
```bash
docker compose -f docker-compose.ee.yaml up -d
```

### 4. Verify Setup

Check that Kong is running:
```bash
curl -i http://localhost:8001/status
```

## Usage

### 🚀 Recommended: Interactive Demo Application

The **preferred way to use** this Kong AI gateway setup is with the **AI Voice-to-Image Demo** application:

🎯 **Repository:** [https://github.com/hguerrero/ai-voice-to-image-demo](https://github.com/hguerrero/ai-voice-to-image-demo)

This demo application provides the **complete user experience**:
- **Voice Input:** Record audio prompts using your microphone
- **Audio Transcription:** Converts speech to text using the Whisper API through Kong
- **Image Generation:** Creates images from transcribed prompts using DALL-E 3 through Kong
- **Interactive UI:** Web-based interface for the complete workflow
- **Real-world Usage:** Demonstrates practical integration patterns

**Why use this demo?**
- ✅ Uses both AI endpoints (transcription + image generation) in one workflow
- ✅ Provides immediate visual feedback
- ✅ Simulates real user interactions
- ✅ No need to manually craft API requests
- ✅ Perfect for demonstrations and proof-of-concepts

### Alternative: Direct API Usage

If you prefer to integrate directly with the APIs, you can use these endpoints:

#### Image Generation

Generate an image using DALL-E 3:

```bash
curl -X POST http://localhost:8000/ai/images \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "A futuristic cityscape with flying cars at sunset",
    "n": 1,
    "size": "1024x1024",
    "quality": "standard"
  }'
```

Expected response:
```json
{
  "created": 1699999999,
  "data": [
    {
      "url": "https://oaidalleapiprodscus.blob.core.windows.net/...",
      "revised_prompt": "A futuristic cityscape..."
    }
  ]
}
```

#### Audio Transcription

Transcribe an audio file using Whisper:

```bash
curl -X POST http://localhost:8000/ai/transcriptions \
  -H "Content-Type: multipart/form-data" \
  -F "file=@/path/to/your/audio.mp3" \
  -F "model=whisper-1"
```

Expected response:
```json
{
  "text": "The transcribed text from your audio file."
}
```

## Configuration Details

### AI Proxy Advanced Plugin

The Kong configuration includes two AI Proxy Advanced plugin instances:

**Image Generation Service:**
- **Route:** `/ai/images`
- **Provider:** OpenAI
- **Model:** DALL-E 3
- **Category:** `image/generation`
- **Features:** Load balancing, retries, payload logging

**Audio Transcription Service:**
- **Route:** `/ai/transcriptions`
- **Provider:** OpenAI
- **Model:** Whisper-1
- **Category:** `audio/transcription`
- **Features:** Large file support (100MB), load balancing, retries

### Key Features

1. **Load Balancing:** Round-robin algorithm with token counting
2. **Retries:** Automatic retry on failures (up to 3 attempts)
3. **Logging:** Full request/response payload logging
4. **CORS:** Enabled for web application integration
5. **Authentication:** Centralized API key management

## Testing

You can test the Kong AI gateway setup using the following curl commands:

### Test Image Generation

```bash
# Simple image generation
curl -X POST http://localhost:8000/ai/images \
  -H "Content-Type: application/json" \
  -d '{"prompt": "A red apple on a wooden table", "size": "512x512"}'

# High-quality image generation
curl -X POST http://localhost:8000/ai/images \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "A professional headshot of a business person",
    "size": "1024x1024",
    "quality": "hd"
  }'
```

### Test Audio Transcription

```bash
# Create a test audio file (requires ffmpeg)
echo "Hello, this is a test audio message" | \
  espeak --stdout | \
  ffmpeg -f wav -i - -f mp3 test_audio.mp3

# Transcribe the audio
curl -X POST http://localhost:8000/ai/transcriptions \
  -F "file=@test_audio.mp3" \
  -F "model=whisper-1"
```

## Monitoring

### Check Kong Logs

```bash
docker logs kong-docker
```

### Monitor AI Requests

The AI Proxy Advanced plugin logs detailed information about:
- Request/response payloads
- Token usage
- Latency metrics
- Error rates

## Troubleshooting

### Common Issues

1. **401 Unauthorized**
   - Verify your OpenAI API key is correctly set in `kong-config/kong.yaml`
   - Ensure the API key has sufficient credits

2. **Plugin Not Found**
   - Verify you're using Kong Enterprise or Konnect
   - Check that the AI Proxy Advanced plugin is available

3. **Large File Upload Issues**
   - Audio files are limited to 100MB
   - Ensure your audio file is in a supported format (mp3, wav, m4a, etc.)

4. **Connection Issues**
   - Verify Docker containers are running: `docker ps`
   - Check Kong status: `curl http://localhost:8001/status`

### Useful Commands

```bash
# Check running containers
docker ps

# View Kong configuration
curl http://localhost:8001/config

# Check plugin status
curl http://localhost:8001/plugins

# Restart Kong
docker restart kong-docker
```

## Cleanup

Stop and remove all containers:

```bash
docker compose down
```

Remove volumes (if needed):

```bash
docker compose down -v
```

## Next Steps

- Explore different DALL-E 3 parameters (style, quality, size)
- Implement rate limiting for AI requests
- Add authentication and authorization
- Set up monitoring and alerting for AI usage
- Integrate with your applications using the Kong Admin API

## Additional Resources

- [Kong AI Proxy Advanced Plugin Documentation](https://docs.konghq.com/hub/kong-inc/ai-proxy-advanced/)
- [OpenAI API Documentation](https://platform.openai.com/docs/api-reference)
- [Kong Enterprise Documentation](https://docs.konghq.com/gateway/latest/)
- [Kong Konnect Documentation](https://docs.konghq.com/konnect/)
