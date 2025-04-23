# Request Callout Example

This example demonstrates how to use Kong's Request Callout plugin to enrich API requests with data from multiple backend services.

## Overview

The setup includes:
- A main service that receives requests and enriches them with user and address data
- A mock service that simulates user and address backend APIs
- Two callouts that fetch data sequentially (user data first, then address data)

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- Kong Enterprise license (for the Request Callout plugin)

## Setup

1. Start the Kong Gateway:
```sh
docker compose up -d
```

2. Verify the services are running:
```sh
curl localhost:8001/services
```

## Testing the Setup

Send a request with a user ID:

```sh
curl -X POST http://localhost:8000 \
  -H "Host: callout.dev" \
  -H "Content-Type: application/json" \
  -d '{"id": "123"}'
```

Expected response:
```json
{
  "username": "John Doe",
  "country": "USA"
}
```

## How It Works

1. When a request arrives with a user ID, the first callout (`c1`) fetches user data:
   - Calls `/v1/users/{id}`
   - Returns user information including username and address_id

2. The second callout (`c2`) uses the address_id from the first callout:
   - Calls `/v1/addresses/{id}`
   - Returns address information including country

3. The final response combines:
   - Username from the first callout
   - Country from the second callout

## Mock API

The example includes a mock API (using Kong's Mocking plugin) that simulates two backend services:

- User Service (`/v1/users/{id}`):
  - Returns user details including username and address_id
  - Example users: 123 (John Doe), 124 (Jane Smith)

- Address Service (`/v1/addresses/{id}`):
  - Returns address details including country
  - Example addresses: 456 (USA), 457 (Germany)

## Configuration Files

- `kong-config/kong.yaml`: Main Kong configuration including services, routes, and plugins
- `mock-api.yaml`: OpenAPI specification for the mock backend services
- `docker-compose.yaml`: Docker Compose configuration for running Kong

## Additional Notes

- The Request Callout plugin requires Kong Enterprise
- The mock service runs on the same Kong instance using the Mocking plugin
- Callouts are executed sequentially, with `c2` depending on the response from `c1`