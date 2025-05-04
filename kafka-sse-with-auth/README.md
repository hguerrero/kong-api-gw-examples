# Kafka SSE with Authentication Example

This example demonstrates how to use Kong API Gateway to consume Kafka messages via Server-Sent Events (SSE) with OpenID Connect (OIDC) authentication.

## Overview

The setup includes:
- Kong API Gateway
- Apache Kafka
- Keycloak for authentication
- A Kafka consumer route with SSE and OIDC plugins

## Prerequisites

- Docker and Docker Compose
- Kong Enterprise license (for the OIDC plugin)
- `curl` for testing
- `kcat` for producing Kafka messages

## Setup

1. Start the Docker Compose environment:
   ```sh
   docker compose up -d
   ```

2. Wait for all services to be healthy.

## Usage

### Authenticate

1. Get an access token from Keycloak:
   ```sh
   TOKEN=$(curl -X POST http://localhost:8080/realms/kafka-realm/protocol/openid-connect/token \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "grant_type=password" \
     -d "client_id=kafka-client" \
     -d "client_secret=secret123" \
     -d "username=kafka" \
     -d "password=kafka" | jq -r '.access_token')
   ```

### Consume Messages

2. Use the token to consume messages via SSE:
   ```sh
   curl -H "Authorization: Bearer $TOKEN" -H 'Host: kafka.dev' http://localhost:8000/kafka/consumer/sse
   ```

### Produce Messages

3. In another terminal, produce a message to the Kafka topic:
   ```sh
   echo '{"message": "Hello, authenticated world!"}' | kcat -P -b localhost:9092 -t streaming
   ```

You should see the message appear in the SSE stream from step 2.

## Configuration

- `kong-config/kong.yaml`: Contains the Kong configuration, including the Kafka consumer and OIDC plugins.
- `realm-export.json`: Keycloak realm configuration with pre-configured client and user.

## Security Considerations

- This example uses a pre-configured Keycloak realm for simplicity. In production, use proper security measures.
- The OIDC plugin is configured with `verify_signature: false` for testing. Enable signature verification in production.

## Troubleshooting

- If you encounter issues, check the logs of the services:
  ```sh
  docker compose logs kong
  docker compose logs kafka
  docker compose logs keycloak
  ```

- Ensure all services are healthy:
  ```sh
  docker compose ps
  ```

## Cleanup

To stop and remove all containers:
```sh
docker compose down
```

