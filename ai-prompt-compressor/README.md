# AI Prompt Compressor

This example demonstrates how to use Kong's AI Prompt Compressor plugin to optimize LLM requests by intelligently compressing prompts while maintaining their semantic meaning and effectiveness.

## Overview

This setup showcases Kong Gateway's ability to:
- Compress prompts before sending to LLM models
- Reduce token usage and costs
- Maintain semantic meaning during compression
- Integrate with local LLM models
- Provide configurable compression strategies

## Prerequisites

**Software Requirements:**
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [curl](https://curl.se/) (for testing)
- [jq](https://stedolan.github.io/jq/) (for JSON formatting)

**Kong Requirements:**
- Kong Enterprise license (for AI plugins)

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

### 2. Start the Services

**For Konnect:**
```bash
docker compose up -d
```

**For Kong Enterprise:**
```bash
docker compose -f docker-compose.ee.yaml up -d
```

This will start:
- Kong Gateway with AI plugins
- LLMlingua compression service
- Local LLM model (Qwen2.5:0.5b-instruct)

### 3. Verify Setup

Check that all services are running:
```bash
# Check Kong status
curl -i http://localhost:8001/status

# Check compression service
curl -i http://localhost:9000/health

# Check containers
docker ps
```

## Usage

### Basic Chat Request

Send a chat request that will be automatically compressed:

```bash
curl -X POST http://localhost:8000/ai/chat \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "user",
        "content": "Please explain in great detail how machine learning algorithms work, including the mathematical foundations, different types of algorithms like supervised learning, unsupervised learning, and reinforcement learning, and provide examples of real-world applications where these algorithms are commonly used in various industries."
      }
    ],
    "max_tokens": 500,
    "temperature": 0.7
  }'
```

### Test Compression Effectiveness

Compare token usage with and without compression:

```bash
# Long prompt that will trigger compression
LONG_PROMPT="Artificial intelligence and machine learning have revolutionized numerous industries and continue to shape the future of technology. These advanced computational techniques enable computers to learn from data, recognize patterns, and make intelligent decisions without being explicitly programmed for every scenario. Machine learning algorithms can be broadly categorized into supervised learning, where models learn from labeled training data, unsupervised learning, which discovers hidden patterns in unlabeled data, and reinforcement learning, where agents learn through interaction with an environment. Deep learning, a subset of machine learning, utilizes neural networks with multiple layers to process complex data and has achieved remarkable success in areas such as computer vision, natural language processing, and speech recognition."

curl -X POST http://localhost:8000/ai/chat \
  -H "Content-Type: application/json" \
  -d "{
    \"messages\": [
      {
        \"role\": \"user\",
        \"content\": \"$LONG_PROMPT Please summarize the key points.\"
      }
    ],
    \"max_tokens\": 200
  }" | jq
```

## Configuration Details

### AI Prompt Compressor Plugin

The compression plugin is configured with:

**Compression Strategy:**
- **Type:** `target_token` - Compresses based on token count thresholds
- **Service URL:** `http://kong-compressor:8080`
- **Error Handling:** Stops on compression errors

**Compression Ranges:**
- **1-50 tokens:** Compress to 25 tokens (50% reduction)
- **50-100 tokens:** Compress to 75 tokens (25% reduction)
- **100-1000 tokens:** Compress to 80 tokens (92% reduction)

### AI Proxy Advanced Plugin

The AI proxy is configured with:
- **Provider:** OpenAI-compatible API
- **Model:** Qwen2.5:0.5b-instruct (local model)
- **Endpoint:** Local LLM service
- **Max Tokens:** 1024
- **Temperature:** 0.5

### LLMlingua Compression Service

The compression service uses:
- **Device:** CPU (configurable to GPU)
- **Algorithm:** LLMlingua for intelligent compression
- **Logging:** Debug level for detailed insights

## Testing Scenarios

### 1. Short Prompt (No Compression)

```bash
curl -X POST http://localhost:8000/ai/chat \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### 2. Medium Prompt (Moderate Compression)

```bash
curl -X POST http://localhost:8000/ai/chat \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "user", 
        "content": "Can you explain the basic concepts of machine learning and provide a simple example of how it works in practice?"
      }
    ]
  }'
```

### 3. Long Prompt (Heavy Compression)

```bash
curl -X POST http://localhost:8000/ai/chat \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {
        "role": "user",
        "content": "I need a comprehensive explanation of artificial intelligence, machine learning, deep learning, neural networks, natural language processing, computer vision, reinforcement learning, supervised learning, unsupervised learning, data preprocessing, feature engineering, model training, model evaluation, overfitting, underfitting, cross-validation, hyperparameter tuning, and the various applications of these technologies in healthcare, finance, automotive, retail, and entertainment industries."
      }
    ]
  }'
```

## Monitoring and Debugging

### Check Compression Logs

```bash
# Kong logs (includes compression details)
docker logs kong-docker

# Compression service logs
docker logs kong-compressor
```

### Monitor Token Usage

The AI Prompt Compressor plugin logs:
- Original prompt token count
- Compressed prompt token count
- Compression ratio
- Processing time

### Compression Service Health

```bash
# Check service health
curl http://localhost:9000/health

# Test compression directly
curl -X POST http://localhost:9000/compress \
  -H "Content-Type: application/json" \
  -d '{
    "text": "This is a long text that should be compressed to save tokens and reduce costs.",
    "target_token": 10
  }'
```

## Performance Optimization

### Compression Tuning

Adjust compression ranges in `kong-config/kong.yaml`:

```yaml
compression_ranges:
- min_tokens: 1
  max_tokens: 50
  value: 25        # Target token count
- min_tokens: 50
  max_tokens: 100
  value: 75
- min_tokens: 100
  max_tokens: 1000
  value: 80
```

### GPU Acceleration

To use GPU for compression (if available):

```yaml
# In docker-compose.yaml
environment:
  - LLMLINGUA_DEVICE_MAP=cuda  # or 'auto'
```

## Troubleshooting

### Common Issues

1. **Compression Service Not Starting**
   ```bash
   # Check logs
   docker logs kong-compressor
   
   # Verify port availability
   netstat -tulpn | grep 9000
   ```

2. **High Compression Latency**
   - Consider GPU acceleration
   - Adjust compression ranges
   - Monitor service resources

3. **Compression Errors**
   ```bash
   # Check compression service health
   curl http://localhost:9000/health
   
   # Test compression manually
   curl -X POST http://localhost:9000/compress \
     -H "Content-Type: application/json" \
     -d '{"text": "test", "target_token": 5}'
   ```

4. **Plugin Configuration Issues**
   ```bash
   # Verify plugin configuration
   curl http://localhost:8001/plugins
   
   # Check service configuration
   curl http://localhost:8001/services
   ```

## Cleanup

Stop all services:

```bash
docker compose down
```

Remove volumes and images:

```bash
docker compose down -v --rmi all
```

## Advanced Configuration

### Custom Compression Models

To use different compression models, modify the compression service configuration or deploy alternative compression services.

### Integration with External LLMs

Update the AI Proxy Advanced configuration to point to external LLM providers:

```yaml
upstream_url: https://api.openai.com/v1/chat/completions
# Add appropriate authentication
```

## Next Steps

- Experiment with different compression ratios
- Integrate with production LLM services
- Implement cost monitoring and alerting
- Add authentication and rate limiting
- Scale compression service for high throughput

## Additional Resources

- [Kong AI Prompt Compressor Plugin Documentation](https://docs.konghq.com/hub/kong-inc/ai-prompt-compressor/)
- [Kong AI Proxy Advanced Plugin Documentation](https://docs.konghq.com/hub/kong-inc/ai-proxy-advanced/)
- [LLMlingua Documentation](https://github.com/microsoft/LLMLingua)
- [Kong Enterprise Documentation](https://docs.konghq.com/gateway/latest/)
