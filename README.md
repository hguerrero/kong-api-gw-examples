# Kong API Gateway Examples

This repository contains practical examples demonstrating various features and use cases of Kong API Gateway.

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
kong-examples/
├── README.md
├── request-callout/
│   ├── README.md
│   ├── docker-compose.yaml
│   ├── kong-config/
│   │   └── kong.yaml
│   └── mock-api.yaml
├── kafka-http-mediation/
│   ├── README.md
│   ├── docker-compose.yaml
│   └── kong-config/
│       └── kong.yaml
└── kafka-sse-with-auth/
    ├── README.md
    ├── docker-compose.yaml
    ├── kong-config/
    │   └── kong.yaml
    └── realm-export.json
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

- [Kong Documentation](https://docs.konghq.com/)
- [Kong Enterprise](https://konghq.com/products/kong-enterprise)
- [Kong Nation Community](https://discuss.konghq.com/)
