# Solace HTTP Mediation

This example demonstrates how to use Kong API Gateway to mediate between HTTP and Solace PubSub+ messaging, enabling seamless integration between web applications and enterprise messaging systems.

## Overview

This setup showcases Kong Gateway's ability to:
- Convert HTTP requests to Solace messages
- Route messages to specific queues and topics
- Handle authentication with Solace brokers
- Transform message payloads
- Provide persistent message delivery

## Prerequisites

**Software Requirements:**
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [curl](https://curl.se/) (for testing)

**Kong Requirements:**
- Kong Enterprise license (for Solace Upstream plugin)

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
- Solace PubSub+ Standard broker
- Kong Gateway with Solace Upstream plugin

### 3. Wait for Services to Initialize

Solace PubSub+ takes a few minutes to fully initialize:

```bash
# Check Solace status
docker logs solace

# Wait for "Local Status: AD-Active" message
# This indicates Solace is ready
```

### 4. Verify Setup

```bash
# Check Kong status
curl -i http://localhost:8001/status

# Check Solace management interface
curl -i http://localhost:8080
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
   - **Username:** gwuser
   - **Password:** gwpassword
   - **Client Profile:** default
   - **ACL Profile:** default

### Create Queue

1. Navigate to **Queues**
2. Create a new queue:
   - **Queue Name:** solace.test
   - **Access Type:** Exclusive
   - **Permission:** Consume, Modify-Topic, Delete

## Usage

### Send Messages to Solace

Send HTTP POST requests that will be converted to Solace messages:

```bash
curl -X POST http://localhost:8000/solace/producer \
  -H "Content-Type: application/json" \
  -d '{
    "message": "Hello from Kong to Solace!",
    "timestamp": "2024-01-15T10:30:00Z",
    "source": "kong-gateway"
  }'
```

### Send Plain Text Messages

```bash
curl -X POST http://localhost:8000/solace/producer \
  -H "Content-Type: text/plain" \
  -d "Simple text message to Solace queue"
```

### Send Complex JSON Payloads

```bash
curl -X POST http://localhost:8000/solace/producer \
  -H "Content-Type: application/json" \
  -d '{
    "order": {
      "id": "12345",
      "customer": "John Doe",
      "items": [
        {"product": "Widget A", "quantity": 2, "price": 19.99},
        {"product": "Widget B", "quantity": 1, "price": 29.99}
      ],
      "total": 69.97
    },
    "metadata": {
      "source": "web-store",
      "timestamp": "2024-01-15T10:30:00Z"
    }
  }'
```

## Monitoring Messages

### Using Solace PubSub+ Manager

1. Navigate to **Queues** → **solace.test**
2. Click on the queue to view details
3. Go to **Messages** tab to see queued messages
4. Use **Message Browser** to inspect message content

### Using Solace CLI (Optional)

```bash
# Connect to Solace container
docker exec -it solace /usr/sw/loads/currentload/bin/cli

# In the CLI:
enable
configure
message-spool message-vpn default
show queue solace.test detail
show queue solace.test messages
```

## Configuration Details

### Solace Upstream Plugin

The Kong configuration includes:

**Connection Settings:**
- **Host:** `tcp://solace:55555` (SMF protocol)
- **VPN:** default
- **Authentication:** Basic (gwuser/gwpassword)

**Message Settings:**
- **Destination:** solace.test (Queue)
- **Delivery Mode:** PERSISTENT
- **Body Forwarding:** Enabled
- **Message Function:** `return message.body`

### Message Transformation

The plugin supports Lua functions for message transformation:

```lua
-- Current configuration
return message.body

-- Example: Add timestamp
return {
  body = message.body,
  timestamp = os.time(),
  source = "kong-gateway"
}

-- Example: Transform JSON
local json = require("cjson")
local data = json.decode(message.body)
data.processed_by = "kong"
data.processed_at = os.time()
return json.encode(data)
```

## Testing Scenarios

### 1. Basic Message Flow

```bash
# Send a simple message
curl -X POST http://localhost:8000/solace/producer \
  -H "Content-Type: application/json" \
  -d '{"test": "basic message flow"}'

# Check in Solace Manager: Queues → solace.test → Messages
```

### 2. High Volume Testing

```bash
# Send multiple messages
for i in {1..10}; do
  curl -X POST http://localhost:8000/solace/producer \
    -H "Content-Type: application/json" \
    -d "{\"message\": \"Test message $i\", \"id\": $i}"
  sleep 1
done
```

### 3. Error Handling

```bash
# Test with invalid JSON
curl -X POST http://localhost:8000/solace/producer \
  -H "Content-Type: application/json" \
  -d '{"invalid": json}'

# Check Kong logs for error handling
docker logs kong-docker
```

## Monitoring and Debugging

### Kong Logs

```bash
# View Kong logs
docker logs kong-docker

# Follow logs in real-time
docker logs -f kong-docker
```

### Solace Logs

```bash
# View Solace logs
docker logs solace

# Monitor Solace events
docker logs -f solace
```

### Solace Metrics

Access Solace monitoring at:
- **PubSub+ Manager:** http://localhost:8080
- **SEMP API:** http://localhost:8080/SEMP/v2/monitor

### Kong Admin API

```bash
# Check plugin status
curl http://localhost:8001/plugins

# View route configuration
curl http://localhost:8001/routes

# Check service health
curl http://localhost:8001/services
```

## Advanced Configuration

### Multiple Destinations

Configure multiple queues/topics:

```yaml
message:
  destinations:
  - name: solace.orders
    type: QUEUE
  - name: solace.notifications
    type: TOPIC
```

### Message Routing

Route messages based on content:

```yaml
functions:
- |
  if message.headers["content-type"] == "application/json" then
    return {
      destination = "solace.json.queue",
      body = message.body
    }
  else
    return {
      destination = "solace.text.queue", 
      body = message.body
    }
  end
```

### Authentication Options

Configure different authentication methods:

```yaml
authentication:
  scheme: BASIC  # or CLIENT_CERTIFICATE, KERBEROS
  username: gwuser
  password: gwpassword
```

## Troubleshooting

### Common Issues

1. **Connection Refused**
   ```bash
   # Check if Solace is running
   docker ps | grep solace
   
   # Verify Solace ports
   netstat -tulpn | grep 55555
   ```

2. **Authentication Failed**
   - Verify user exists in Solace PubSub+ Manager
   - Check username/password in Kong configuration
   - Ensure user has proper permissions

3. **Queue Not Found**
   - Create queue in Solace PubSub+ Manager
   - Verify queue name matches configuration
   - Check queue permissions

4. **Plugin Not Loading**
   ```bash
   # Verify Kong Enterprise license
   curl http://localhost:8001/licenses
   
   # Check available plugins
   curl http://localhost:8001/plugins/enabled
   ```

### Useful Commands

```bash
# Restart services
docker compose restart

# View all containers
docker ps -a

# Check network connectivity
docker exec kong-docker ping solace

# Test Solace connectivity
docker exec kong-docker telnet solace 55555
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

## Next Steps

- Configure message consumers
- Implement request-reply patterns
- Add message transformation logic
- Set up monitoring and alerting
- Integrate with enterprise applications
- Explore Solace topics and subscriptions

## Additional Resources

- [Kong Solace Upstream Plugin Documentation](https://docs.konghq.com/hub/kong-inc/solace-upstream/)
- [Solace PubSub+ Documentation](https://docs.solace.com/)
- [Solace Developer Portal](https://solace.dev/)
- [Kong Enterprise Documentation](https://docs.konghq.com/gateway/latest/)
