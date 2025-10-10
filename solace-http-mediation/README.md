# Solace HTTP Mediation

This example demonstrates how to use Kong API Gateway to mediate between HTTP and Solace PubSub+ messaging, enabling seamless integration between web applications and enterprise messaging systems.

## Overview

This setup showcases Kong Gateway's ability to:
- Convert HTTP requests to Solace messages
- Consume messages from Solace queues via HTTP GET requests
- Route messages to specific queues and topics using dynamic routing
- Handle authentication with Solace brokers
- Transform and validate message payloads
- Provide security and rate limiting

## Prerequisites

**Software Requirements:**
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [curl](https://curl.se/) (for testing)

**Kong Requirements:**
- Kong Enterprise license (for Solace plugins)
- Kong Konnect account (recommended) or Kong Enterprise deployment

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

### 2. Configure Kong Secrets

Before starting, you need to configure the Solace credentials in Kong's secret vault:

**For Konnect:**
1. Login to Kong Konnect
2. Navigate to Gateway Manager → Secrets
3. Create secrets:
   - `solace-username`: `<SOLACE_USERNAME>`
   - `solace-password`: `<SOLACE_PASSWORD>`

**For Kong Enterprise:**
Update the kong.yaml file to replace vault references with actual credentials or configure Kong's vault system.

### 3. Update Configuration

Replace placeholders in `kong-config/kong.yaml`:
- `<SOLACE_HOST>`: Your Solace broker hostname (e.g., `mr-connection-xxxxx.messaging.solace.cloud`)
- `<SOLACE_VPN>`: Your Solace VPN name (e.g., `demo`)
- `<CONFIG_STORE_ID>`: Your Konnect configuration store ID

### 4. Start the Services

**For Konnect:**
```bash
docker compose up -d
```

**For Kong Enterprise:**
```bash
docker compose -f docker-compose.ee.yaml up -d
```

### 5. Wait for Services to Initialize

Kong takes a moment to sync configuration from Konnect:

```bash
# Check Kong status
curl -i http://localhost:8001/status

# Verify configuration sync
curl -i http://localhost:8001/routes
```

## Solace Configuration

### Access Solace PubSub+ Manager

1. Open your browser to http://localhost:8080
2. Login with:
   - **Username:** admin
   - **Password:** admin

### Create Message VPN User

1. Navigate to **Message VPNs** → **default**
2. Go to **Access Control** → **Client Usernames**
3. Create a new user:
   - **Username:** <SOLACE_USERNAME>
   - **Password:** <SOLACE_PASSWORD>
   - **Client Profile:** default
   - **ACL Profile:** default

### Create Queues for Message Consumption

Create queues that will be used by the consume plugin to receive messages:

1. Navigate to **Queues**
2. Create queues for each topic you want to consume from:

   **For orders/processed topic:**
   - **Queue Name:** orders.processed
   - **Access Type:** Exclusive
   - **Permission:** Consume, Modify-Topic, Delete

   **For notifications/status topic:**
   - **Queue Name:** notifications.status
   - **Access Type:** Exclusive
   - **Permission:** Consume, Modify-Topic, Delete

### Configure Topic Subscriptions

For each queue, add topic subscriptions to receive messages from the corresponding topics:

1. Navigate to the queue (e.g., **orders.processed**)
2. Go to **Subscriptions** tab
3. Add topic subscriptions:

   **For orders.processed queue:**
   - Add subscription: `orders/processed`

   **For notifications.status queue:**
   - Add subscription: `notifications/status`

### Configure ACL Profile (Optional)

To ensure proper access control, you may need to configure the ACL profile:

1. Navigate to **Access Control** → **ACL Profiles** → **default**
2. Go to **Publish Topic Exceptions**
3. Add exceptions for topics your application will publish to:
   - `orders/>`
   - `notifications/>`
4. Go to **Subscribe Topic Exceptions**  
5. Add exceptions for topics your application will subscribe to:
   - `orders/>`
   - `notifications/>`

### Verify Configuration

Test that the configuration is working:

1. Use the **Try Me!** tab in PubSub+ Manager
2. **Publisher:**
   - Connect to the broker
   - Publish a test message to `orders/new`
3. **Subscriber:**
   - Connect to the broker
   - Subscribe to queue `orders.processed`
   - Verify you can receive messages

## Usage

### Send Messages to Solace (HTTP → Solace)

Send HTTP POST requests that will be converted to Solace messages and published to topics:

```bash
# Send order to orders/new topic
curl -X POST "http://localhost:8000/solace/orders/new" \
  -H "apikey: internal1234" \
  -H "Content-Type: application/json" \
  -d '{
    "orderId": "ORD-001",
    "customerId": "CUST-123",
    "items": [
      {"product": "Widget A", "quantity": 2, "price": 19.99}
    ],
    "total": 39.98
  }'
```

```bash
# Send notification to notifications/alerts topic
curl -X POST "http://localhost:8000/solace/notifications/alerts" \
  -H "apikey: partner1234" \
  -H "Content-Type: application/json" \
  -d '{
    "orderId": "ORD-002",
    "customerId": "CUST-456",
    "items": [
      {"product": "Widget B", "quantity": 1, "price": 29.99}
    ],
    "total": 29.99
  }'
```

### Consume Messages from Solace (Solace → HTTP)

Retrieve messages from Solace topics via HTTP GET requests:

```bash
# Consume from orders/processed topic
curl -X GET "http://localhost:8000/solace/orders/processed" \
  -H "apikey: internal1234"
```

```bash
# Consume from notifications/status topic
curl -X GET "http://localhost:8000/solace/notifications/status" \
  -H "apikey: partner1234"
```

### Dynamic Topic Routing

The configuration uses URI path segments to dynamically determine Solace topics:

- `POST /solace/orders/new` → publishes to topic `orders/new`
- `POST /solace/events/user-signup` → publishes to topic `events/user-signup`
- `GET /solace/orders/processed` → consumes from topic `orders/processed`

## Security Features

### Authentication Methods

The API supports multiple authentication methods:

1. **API Key Authentication** (`key-auth` plugin)
   - Header: `apikey`
   - Users: `partner1234`, `internal1234`

2. **JWT Authentication** (`jwt` plugin)
   - Header: `authorization`
   - Algorithm: HS256
   - Claims validation: `exp`, `nbf`

### Access Control

**ACL Groups:**
- `anonymous`: No access to protected resources
- `solace`: Can access incoming orders route
- `admin`: Full access to all routes

**Route-specific permissions:**
- Incoming orders route: Requires `solace` or `admin` group
- Service-level: Denies `anonymous` group access

### Rate Limiting

- **Limit:** 5 requests per minute per client
- **Policy:** Local (in-memory)

### Request Size Limiting

- **Maximum payload:** 8KB (8192 bytes)
- **Applied to:** All requests

## Message Validation

Incoming order messages are validated against a JSON schema:

**Required fields:**
- `orderId` (string)
- `customerId` (string) 
- `items` (array)
- `total` (number)

**Example valid payload:**
```json
{
  "orderId": "ORD-001",
  "customerId": "CUST-123",
  "items": [
    {"product": "Widget A", "quantity": 2, "price": 19.99}
  ],
  "total": 39.98
}
```

## Message Transformation

### Request Transformation

All incoming messages are automatically enriched with:
```json
{
  "source": "kong-event-driven-api"
}
```

### Solace Message Configuration

**For Publishing (HTTP → Solace):**
- **Delivery Mode:** DIRECT
- **Topic:** Dynamic based on URI path
- **Body:** HTTP request body forwarded as-is
- **Function:** `return message.body`

**For Consuming (Solace → HTTP):**
- **Mode:** WEBSOCKET
- **Binding:** Dynamic based on URI path
- **Flow:** Configured to bind to requested topic

## Testing Scenarios

### 1. Complete Order Flow

```bash
# 1. Send new order
curl -X POST "http://localhost:8000/solace/orders/new" \
  -H "apikey: internal1234" \
  -H "Content-Type: application/json" \
  -d '{
    "orderId": "ORD-001",
    "customerId": "CUST-123",
    "items": [{"product": "Widget", "quantity": 1, "price": 25.00}],
    "total": 25.00
  }'

# 2. Check for processed orders
curl -X GET "http://localhost:8000/solace/orders/processed" \
  -H "apikey: internal1234"
```

### 2. Authentication Testing

```bash
# Test anonymous access (should fail)
curl -X POST "http://localhost:8000/solace/orders/new" \
  -H "Content-Type: application/json" \
  -d '{"test": "data"}'

# Test with valid API key
curl -X POST "http://localhost:8000/solace/orders/new" \
  -H "apikey: internal1234" \
  -H "Content-Type: application/json" \
  -d '{
    "orderId": "ORD-002",
    "customerId": "CUST-456", 
    "items": [],
    "total": 0
  }'
```

### 3. Rate Limiting Testing

```bash
# Send multiple requests quickly to trigger rate limiting
for i in {1..10}; do
  curl -X POST "http://localhost:8000/solace/orders/new" \
    -H "apikey: internal1234" \
    -H "Content-Type: application/json" \
    -d "{\"orderId\": \"ORD-$i\", \"customerId\": \"CUST-$i\", \"items\": [], \"total\": 0}"
  echo "Request $i sent"
done
```

### 4. Validation Testing

```bash
# Test invalid payload (missing required fields)
curl -X POST "http://localhost:8000/solace/orders/new" \
  -H "apikey: internal1234" \
  -H "Content-Type: application/json" \
  -d '{"invalid": "payload"}'

# Test valid payload
curl -X POST "http://localhost:8000/solace/orders/new" \
  -H "apikey: internal1234" \
  -H "Content-Type: application/json" \
  -d '{
    "orderId": "ORD-VALID",
    "customerId": "CUST-VALID",
    "items": [{"product": "Test", "quantity": 1}],
    "total": 10.00
  }'
```

## Monitoring and Debugging

### Kong Admin API

```bash
# Check service status
curl http://localhost:8001/services

# View routes configuration
curl http://localhost:8001/routes

# Check plugin configuration
curl http://localhost:8001/plugins

# Monitor consumer access
curl http://localhost:8001/consumers
```

### Kong Logs

```bash
# View Kong logs
docker logs kong-gateway

# Follow logs in real-time  
docker logs -f kong-gateway
```

### Health Checks

```bash
# Kong health check
curl -i http://localhost:8001/status

# Service connectivity check
curl -i http://localhost:8000/solace/orders/new \
  -H "apikey: internal1234" \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{"orderId":"HEALTH","customerId":"CHECK","items":[],"total":0}'
```

## Configuration Customization

### Adding New Topics

To add support for new topics, the current configuration automatically supports any topic via the regex pattern:

- **Incoming:** `~/solace/(?<topic_id>[\w/-]+)` - supports topics like `orders/new`, `events/user-signup`
- **Outgoing:** `~/solace/(?<topic_id>[\w-]+)` - supports topics like `orders-processed`, `notifications-status`

### Adding New Consumers

Add new consumers to the configuration:

```yaml
consumers:
  - username: <NEW_USERNAME>
    acls:
      - group: <GROUP_NAME>
    keyauth_credentials:
      - key: <API_KEY>
```

### Modifying Authentication

Update JWT configuration:

```yaml
jwt_secrets:
  - algorithm: HS256
    key: <JWT_KEY>
    secret: <JWT_SECRET>
```

## Troubleshooting

### Common Issues

1. **Authentication Failed**
   - Verify API key in request header
   - Check consumer configuration
   - Ensure user has proper ACL group membership

2. **Rate Limited**
   - Wait for rate limit window to reset (1 minute)
   - Consider adjusting rate limiting configuration

3. **Validation Failed**
   - Ensure JSON payload includes all required fields
   - Check payload size (must be under 8KB)
   - Verify JSON format is valid

4. **Solace Connection Issues**
   - Verify Solace credentials in vault/configuration
   - Check Solace host connectivity
   - Ensure VPN name is correct

### Debug Commands

```bash
# Test connectivity
curl -v http://localhost:8001/status

# Check plugin status
curl http://localhost:8001/plugins | jq '.data[] | select(.name | contains("solace"))'

# View consumer details
curl http://localhost:8001/consumers/<USERNAME>

# Check ACL assignments
curl http://localhost:8001/consumers/<USERNAME>/acls
```

## Cleanup

Stop all services:

```bash
docker compose down
```

Remove volumes:

```bash
docker compose down -v
```

## Configuration Placeholders

When setting up this example, replace the following placeholders:

### In kong.yaml:
- `<SOLACE_HOST>`: Your Solace broker hostname
- `<SOLACE_VPN>`: Your Solace VPN name
- `<CONFIG_STORE_ID>`: Your Konnect configuration store ID

### In Konnect Secrets:
- `<SOLACE_USERNAME>`: Your Solace username
- `<SOLACE_PASSWORD>`: Your Solace password

### For Testing:
- `<NEW_USERNAME>`: Additional consumer usernames
- `<GROUP_NAME>`: ACL group names
- `<API_KEY>`: Consumer API keys
- `<JWT_KEY>`: JWT key identifier
- `<JWT_SECRET>`: JWT signing secret
- `<YOUR_JWT_TOKEN>`: Valid JWT token for testing

## Next Steps

- Configure message persistence and queues
- Implement request-reply patterns  
- Add message transformation logic
- Set up monitoring and alerting
- Integrate with enterprise applications
- Explore Solace guaranteed messaging features

## Additional Resources

- [Kong Solace Upstream Plugin Documentation](https://docs.konghq.com/hub/kong-inc/solace-upstream/)
- [Kong Solace Consume Plugin Documentation](https://docs.konghq.com/hub/kong-inc/solace-consume/)
- [Solace PubSub+ Documentation](https://docs.solace.com/)
- [Kong Enterprise Documentation](https://docs.konghq.com/gateway/latest/)
- [Kong Konnect Documentation](https://docs.konghq.com/konnect/)
