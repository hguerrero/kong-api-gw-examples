# Kafka Event-Driven API Example

This example demonstrates how to use Kong API Gateway to consume Kafka messages via WebSocket and Server-Sent Events (SSE) connections using the Kafka Consume plugin.

## Overview

The setup includes:
- Kong API Gateway 3.12 (configured as a Konnect data plane)
- Apache Kafka (latest)
- Kafka consumer routes with WebSocket and SSE support

## Prerequisites

- Docker and Docker Compose
- Kong Enterprise license or Konnect account
- WebSocket client for testing (e.g., Insomnia, `wscat`, browser WebSocket tools)
- `kcat` for producing Kafka messages

## Setup

### Option 1: Using Kong Konnect (Default)

1. Start the Docker Compose environment:
   ```sh
   docker compose up -d
   ```

2. Wait for all services to be healthy:
   ```sh
   docker compose ps
   ```

The Kong instance will connect to Konnect as a data plane using the configuration in `konnect.env`.

### Option 2: Using Kong Enterprise (Local)

1. Update the docker-compose.yaml to use the `ee.env` file instead of `konnect.env`:
   ```yaml
   env_file:
     - ee.env
   ```

2. Start the environment:
   ```sh
   docker compose up -d
   ```

## Usage

### Connect to Kafka Consumer

You can connect to the Kafka consumer using either WebSocket or Server-Sent Events (SSE).

#### Option 1: WebSocket Consumer

1. Connect to the Kafka consumer WebSocket endpoint:
   ```sh
   # Using wscat (install with: npm install -g wscat)
   wscat -c ws://localhost:8000/kafka/ws/my-topic
   ```

   Or using Insomnia:
   - Create a new WebSocket request
   - Set URL to: `ws://localhost:8000/kafka/ws/my-topic`
   - Connect to start receiving messages

#### Option 2: Server-Sent Events (SSE) Consumer

1. Connect to the Kafka consumer SSE endpoint:
   ```sh
   # Using curl
   curl http://localhost:8000/kafka/sse/my-topic
   ```

   Or using any HTTP client that supports streaming responses.

### Produce Messages

2. In another terminal, produce messages to the Kafka topic:
   ```sh
   # Single message
   echo '{"message": "Hello, Kafka!"}' | kcat -P -b localhost:9092 -t my-topic

   # Multiple messages
   echo '{"event": "user_login", "user_id": 123}' | kcat -P -b localhost:9092 -t my-topic
   echo '{"event": "order_created", "order_id": 456}' | kcat -P -b localhost:9092 -t my-topic
   ```

You should see the messages appear in real-time in your consumer connection from step 1.

## Configuration

- `kong-config/kong.yaml`: Contains the Kong configuration with the Kafka consume plugin
- `konnect.env`: Konnect data plane configuration (default)
- `ee.env`: Kong Enterprise license configuration (alternative)
- `docker-compose.yaml`: Docker services configuration

### Kong Configuration Details

The Kong configuration includes:

**Service**: `kafka-local`
- URL: `tcp://localhost:9092`

**Routes**:
- `kafka-consumer-ws`: Listening on `/kafka/ws/my-topic` (WebSocket mode)
- `kafka-consumer-sse`: Listening on `/kafka/sse/my-topic` (Server-Sent Events mode)

**Plugins**: Two `kafka-consume` plugins configured:
1. **WebSocket Plugin**:
   - Route: `kafka-consumer-ws`
   - Topic: `my-topic`
   - Mode: `websocket`
   - Bootstrap servers: `localhost:9092`

2. **SSE Plugin**:
   - Route: `kafka-consumer-sse`
   - Topic: `my-topic`
   - Mode: `server-sent-events`
   - Bootstrap servers: `localhost:9092`

## Troubleshooting

- If you encounter issues, check the logs of the services:
  ```sh
  docker compose logs kong
  docker compose logs kafka
  ```

- Ensure all services are healthy:
  ```sh
  docker compose ps
  ```

- Verify Kafka is accessible:
  ```sh
  kcat -L -b localhost:9092
  ```

- Check if the topic exists:
  ```sh
  kcat -L -b localhost:9092 -t my-topic
  ```

## Cleanup

To stop and remove all containers:
```sh
docker compose down
```

