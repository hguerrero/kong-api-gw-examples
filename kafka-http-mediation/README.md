# Kafka HTTP Mediation

This example demonstrates how to use Kong API Gateway to mediate between HTTP and Kafka, enabling HTTP clients to produce and consume Kafka messages through REST endpoints.

## Overview

The setup includes:
- Kong API Gateway 3.12 (configured as a Konnect data plane)
- Apache Kafka (latest)
- HTTP-to-Kafka producer route
- HTTP consumer route for Kafka messages

## Prerequisites

This uses Kong Enterprise Plugins, so you will need a license. You can get a free trial license from [Kong](https://konghq.com/products/kong-konnect/register).

**Software**

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [kcat](https://github.com/edenhill/kcat)
- [jq](https://stedolan.github.io/jq/)

## Setup

Start the docker compose environment:

```sh
docker compose up -d
```

Wait for all services to be healthy:

```sh
docker compose ps
```

## Usage

### Produce Messages

To produce messages to the `my-topic` Kafka topic via HTTP:

```sh
curl -X POST -d '{"message": "Hello World"}' http://localhost:8000/kafka/my-topic
```

This will produce a message to the `my-topic` topic through the `kafka-upstream` plugin.

> Note: You may need to run it twice as the first call creates the topic and the second call produces the message.

Verify the produced messages using kcat:

```sh
kcat -C -b localhost:9092 -t my-topic
```

If you have jq installed, you can pretty print the messages:

```sh
kcat -C -b localhost:9092 -t my-topic -e | jq
```

You should see the message you produced in the output.

### Consume Messages

To consume messages from the `my-topic` Kafka topic via HTTP:

```sh
curl http://localhost:8000/kafka/rest/my-topic
```

If you have `jq` installed, you can pretty print the response:

```sh
curl http://localhost:8000/kafka/rest/my-topic -s | jq
```

The response will be a JSON object containing messages from the topic.

**Example Response (empty topic)**:

```json
{
  "my-topic": {
    "partitions": {
      "0": {
        "errcode": 0,
        "high_watermark": 0,
        "records": {}
      }
    }
  }
}
```

**Produce a test message**:

Run the following command to send a message to the `my-topic` topic using kcat:

```sh
echo '{"message": "Hello World"}' | kcat -P -b localhost:9092 -t my-topic
```

**Consume the message**:

Now run the consume command again and you should see the message in the response:

```sh
curl http://localhost:8000/kafka/rest/my-topic
```

**Example Response (with message)**:

```json
{
  "my-topic": {
    "partitions": {
      "0": {
        "errcode": 0,
        "high_watermark": 1,
        "records": [
          {
            "key": "",
            "offset": 0,
            "timestamp": 0,
            "value": "{\"message\": \"Hello World\"}"
          }
        ]
      }
    }
  }
}
```

## Configuration

- `kong-config/kong.yaml`: Contains the Kong configuration with Kafka plugins
- `konnect.env`: Konnect data plane configuration (default)
- `ee.env`: Kong Enterprise license configuration (alternative)
- `docker-compose.yaml`: Docker services configuration

### Kong Configuration Details

The Kong configuration includes:

**Service**: `kafka-local`
- URL: `tcp://localhost:9092`

**Routes**:
- `kafka-consumer-http-get`: Listening on `/kafka/rest/my-topic` (HTTP consumer)
- `kafka-producer`: Listening on `/kafka/my-topic` (HTTP producer)

**Plugins**:
1. **Kafka Consume Plugin**:
   - Route: `kafka-consumer-http-get`
   - Topic: `my-topic`
   - Bootstrap servers: `localhost:9092`

2. **Kafka Upstream Plugin**:
   - Route: `kafka-producer`
   - Topic: `my-topic`
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
