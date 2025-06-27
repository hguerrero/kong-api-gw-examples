# Kong Kafka Schema Validation Example

This example demonstrates how to use Kong Gateway as an event gateway with Kafka integration, including schema validation using a Schema Registry. The setup showcases both producing and consuming Kafka messages with Avro schema validation.

## Architecture Overview

The example sets up a complete event-driven architecture with the following components:

- **Kong Gateway**: Acts as an API gateway and event gateway for Kafka operations
- **Apache Kafka**: Message broker for event streaming
- **Schema Registry (Apicurio)**: Manages and validates message schemas (Confluent-compatible)
- **Keycloak**: Identity and access management (for authentication scenarios)
- **Schema Registry UI**: Web interface for managing schemas

## Components

### Services

1. **Kafka** (Port 9092)
   - Apache Kafka 3.9.0 running in KRaft mode
   - Single broker setup for development

2. **Schema Registry** (Port 8080)
   - Apicurio Registry 3.0.9 with Confluent compatibility
   - Manages Avro and JSON schemas

3. **Schema Registry UI** (Port 8888)
   - Web interface for schema management

4. **Keycloak** (Port 18080)
   - Identity provider for authentication
   - Admin credentials: admin/admin

5. **Kong Gateway** (Ports 8000, 8443, 8001, 8002)
   - API Gateway with Kafka plugins
   - Admin API and Manager UI available

### Kong Routes and Plugins

The example configures several routes with Kafka plugins:

#### Producer Route
- **Route**: `/kafka/producer`
- **Plugin**: `kafka-upstream`
- **Features**:
  - Produces messages to `my-topic-incoming` topic
  - Schema validation using Avro schema from registry
  - Synchronous message production
  - Message transformation via Lua functions

#### Consumer Routes
1. **Basic Consumer** (`/kafka/consumer/rest`)
   - REST-based message consumption
   - No schema validation

2. **Schema-Validated Consumer** (`/kafka/consumer/rest/schema`)
   - REST-based consumption with schema validation
   - Deserializes messages using registry schemas

3. **Server-Sent Events Consumer** (`/kafka/consumer/sse`)
   - Real-time message streaming via SSE
   - Schema validation enabled

## Schema Files

### Avro Schema (`avro-schema.json`)
```json
{
  "type": "record",
  "name": "UserRecord",
  "namespace": "kong.avro",
  "fields": [
    {
      "name": "username",
      "type": "string"
    },
    {
      "name": "age",
      "type": "int"
    }
  ]
}
```

### JSON Schema (`json-schema.json`)
```json
{
  "$schema": "https://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "username": {
      "type": "string",
      "minLength": 3
    },
    "age": {
      "type": "integer",
      "minimum": 0
    }
  },
  "required": ["username", "age"]
}
```

## Getting Started

### Prerequisites
- Docker and Docker Compose
- Kong Enterprise license (for EE version)

### Running the Example

#### Option 1: Kong Konnect
```bash
docker-compose up -d
```

#### Option 2: Kong Enterprise
```bash
docker-compose -f docker-compose.ee.yaml up -d
```

### Verify Services
1. **Kong Gateway**: http://localhost:8000
2. **Kong Admin API**: http://localhost:8001
3. **Kong Manager** (EE only): http://localhost:8002
4. **Schema Registry**: http://localhost:8080
5. **Schema Registry UI**: http://localhost:8888
6. **Keycloak**: http://localhost:18080

## Usage Examples

### 1. Register Schema
First, register the Avro schema in the schema registry:

```bash
curl -X POST http://localhost:8080/apis/ccompat/v7/subjects/user-value/versions \
  -H "Content-Type: application/vnd.schemaregistry.v1+json" \
  -d @avro-schema.json
```

### 2. Produce Messages
Send a message through Kong to Kafka with schema validation:

```bash
curl -X POST http://localhost:8000/kafka/producer \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john_doe",
    "age": 30
  }'
```

### 3. Consume Messages

#### REST Consumer (without schema)
```bash
curl http://localhost:8000/kafka/consumer/rest
```

#### REST Consumer (with schema validation)
```bash
curl http://localhost:8000/kafka/consumer/rest/schema
```

#### Server-Sent Events Consumer
```bash
curl -N http://localhost:8000/kafka/consumer/sse
```

## Configuration Files

- `docker-compose.yaml`: Open source Kong setup
- `docker-compose.ee.yaml`: Enterprise Kong setup
- `kong-config/kong.yaml`: Kong declarative configuration
- `konnect.env`: Kong Konnect data plane configuration
- `ee.env`: Kong Enterprise license configuration
- `realm-export.json`: Keycloak realm configuration

## Key Features Demonstrated

1. **Schema Validation**: Automatic validation of Kafka messages using Avro schemas
2. **Multiple Consumer Patterns**: REST and Server-Sent Events consumption
3. **Schema Registry Integration**: Confluent-compatible schema management
4. **Message Transformation**: Lua-based message processing
5. **Authentication Ready**: Keycloak integration for secure access
6. **Enterprise Features**: Kong Manager UI and advanced plugins (EE version)

## Troubleshooting

### Common Issues
1. **Schema Registry Connection**: Ensure schema registry is running before Kong
2. **Topic Creation**: Kafka auto-creates topics, but you can pre-create them if needed
3. **Schema Registration**: Register schemas before producing messages with validation

### Logs
```bash
# View Kong logs
docker logs kong-docker

# View Kafka logs
docker logs kafka

# View Schema Registry logs
docker logs schema-registry
```

## Cleanup
```bash
docker-compose down -v
```

This example provides a foundation for building event-driven architectures with Kong Gateway, demonstrating schema validation, multiple consumption patterns, and integration with modern event streaming platforms.
