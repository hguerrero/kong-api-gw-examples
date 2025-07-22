# Kong API Gateway Examples

This repository contains practical examples demonstrating various features and use cases of Kong API Gateway.

## Table of Contents

### 🤖 AI & Machine Learning
- [AI Image Generation](#ai-image-generation) - OpenAI DALL-E 3 & Whisper integration
- [AI Prompt Compressor](#ai-prompt-compressor) - LLM prompt optimization & compression

### 📨 Event & Message Processing
- [Kafka HTTP Mediation](#kafka-http-mediation) - HTTP ↔ Kafka protocol mediation
- [Kafka SSE with Authentication](#kafka-sse-with-authentication) - Kafka SSE streaming with OIDC
- [Kafka Schema Validation](#kafka-schema-validation) - Event gateway with schema registry
- [Solace HTTP Mediation](#solace-http-mediation) - HTTP ↔ Solace PubSub+ messaging

### 🔧 API Enhancement
- [Request Callout](#request-callout) - API enrichment with backend services

---

## Overview

Each example is self-contained and includes:
- Complete configuration files
- Docker Compose setup
- Detailed documentation
- Step-by-step instructions
- Testing procedures

## Prerequisites

General requirements for all examples:
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [curl](https://curl.se/) (for testing)
- Kong Enterprise license (for enterprise features)

## Examples

### [Request Callout](request-callout)

Demonstrates how to use Kong's Request Callout plugin to enrich API requests with data from multiple backend services.

**Key Features:**
- Sequential API calls
- Request/response transformation
- Mock backend services
- Data enrichment patterns

**Technologies:**
- Kong Request Callout plugin
- Kong Mocking plugin
- OpenAPI specifications

### [Kafka HTTP Mediation](kafka-http-mediation)

Shows how to use Kong API Gateway to mediate between HTTP and Kafka protocols.

**Key Features:**
- HTTP to Kafka message production
- Kafka to HTTP consumption
- Server-Sent Events (SSE) streaming
- Synchronous and asynchronous patterns

**Technologies:**
- Kong Kafka Upstream plugin
- Kong Kafka Consumer plugin
- Apache Kafka
- Server-Sent Events

### [Kafka SSE with Authentication](kafka-sse-with-auth)

Demonstrates how to consume Kafka messages via Server-Sent Events (SSE) with OpenID Connect (OIDC) authentication.

**Key Features:**
- Kafka message consumption via SSE
- OIDC authentication integration
- Keycloak for identity management

**Technologies:**
- Kong Kafka Consumer plugin
- Kong OIDC plugin
- Apache Kafka
- Server-Sent Events

### [Kafka Schema Validation](kafka-schema-validation)

Demonstrates how to use Kong Gateway as an event gateway with Kafka integration, including schema validation using a Schema Registry.

**Key Features:**
- Kafka message production with schema validation
- Multiple consumption patterns (REST and SSE)
- Schema Registry integration
- Avro and JSON schema support

**Technologies:**
- Kong Kafka plugins
- Apache Kafka
- Apicurio Schema Registry (Confluent-compatible)
- Avro schemas

### [AI Image Generation](ai-image-generation)

Demonstrates how to use Kong's AI Proxy Advanced plugin to create an AI gateway for image generation and audio transcription services.

**Key Features:**
- OpenAI DALL-E 3 image generation
- OpenAI Whisper audio transcription
- AI request routing and load balancing
- Token counting and latency optimization
- Request/response logging

**Technologies:**
- Kong AI Proxy Advanced plugin
- OpenAI API (DALL-E 3, Whisper)
- AI request balancing and retries
- CORS support

### [AI Prompt Compressor](ai-prompt-compressor)

Shows how to use Kong's AI Prompt Compressor plugin to optimize LLM requests by compressing prompts while maintaining semantic meaning.

**Key Features:**
- Intelligent prompt compression
- Token-based compression strategies
- LLM request optimization
- Configurable compression ranges
- Integration with local LLM models

**Technologies:**
- Kong AI Prompt Compressor plugin
- Kong AI Proxy Advanced plugin
- LLMlingua compression service
- Local LLM models (Qwen2.5)

### [Solace HTTP Mediation](solace-http-mediation)

Demonstrates how to use Kong API Gateway to mediate between HTTP and Solace PubSub+ messaging.

**Key Features:**
- HTTP to Solace message production
- Queue-based messaging patterns
- Message transformation and routing
- Persistent message delivery
- Authentication integration

**Technologies:**
- Kong Solace Upstream plugin
- Solace PubSub+ Standard
- Message queues and topics
- Basic authentication

## Getting Started

1. Clone this repository:
```sh
git clone https://github.com/your-username/kong-examples.git
cd kong-examples
```

2. Set up your Kong Enterprise license:
```sh
export KONG_LICENSE_DATA='{"license":{"version":1,...}}'
```

3. Choose an example and follow its specific README instructions.

## Project Structure

```
kong-api-gw-examples/
├── README.md
├── ai-image-generation/
│   ├── README.md
│   ├── docker-compose.yaml
│   ├── docker-compose.ee.yaml
│   ├── kong-config/
│   │   └── kong.yaml
│   ├── ee.env
│   └── konnect.env
├── ai-prompt-compressor/
│   ├── README.md
│   ├── docker-compose.yaml
│   ├── docker-compose.ee.yaml
│   ├── kong-config/
│   │   └── kong.yaml
│   ├── ee.env
│   └── konnect.env
├── kafka-http-mediation/
│   ├── README.md
│   ├── docker-compose.yaml
│   └── kong-config/
│       └── kong.yaml
├── kafka-schema-validation/
│   ├── README.md
│   ├── docker-compose.yaml
│   ├── docker-compose.ee.yaml
│   ├── kong-config/
│   │   └── kong.yaml
│   ├── avro-schema.json
│   └── json-schema.json
├── kafka-sse-with-auth/
│   ├── README.md
│   ├── docker-compose.yaml
│   ├── kong-config/
│   │   └── kong.yaml
│   └── realm-export.json
├── request-callout/
│   ├── README.md
│   ├── docker-compose.yaml
│   ├── kong-config/
│   │   └── kong.yaml
│   └── mock-api.yaml
└── solace-http-mediation/
    ├── README.md
    ├── docker-compose.yaml
    ├── docker-compose.ee.yaml
    ├── kong-config/
    │   └── kong.yaml
    ├── ee.env
    └── konnect.env
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

When adding a new example:
1. Create a new directory with a descriptive name
2. Include a comprehensive README
3. Provide all necessary configuration files
4. Add the example to this main README
5. Ensure all code is well-documented

## License

This project is licensed under the [Apache 2.0 License](LICENSE).

## Additional Resources

- [Kong Documentation](https://developer.konghq.com/)
- [Kong Enterprise](https://konghq.com/products/kong-enterprise)
- [Kong Nation Community](https://discuss.konghq.com/)
