# Kafka HTTP Mediation

This example shows how to use Kong API Gateway to mediate between HTTP and Kafka.

## Prerequisites

This uses the Kong Enterprise Plugins, so you will need a license. You can get a free trial license from [Kong](https://konghq.com/products/kong-konnect/register).

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

## Produce

To produce messages to the `incoming` topic run:

```sh
curl -H 'Host: kafka.dev' -X POST -d '{"message": "Hello World"}' http://localhost:8000/kafka/producer
```

This will produce a message to the `incoming` topic.

> Note that you will need to run it twice as the first call creates the topic and the second call produces the message.

Check the produced messages using kcat:

```sh
kcat -C -b localhost:9092 -t incoming
```

If you have jq installed you can use the following to pretty print the messages:

```sh
kcat -C -b localhost:9092 -t incoming -e | jq
```

You should see the message you produced in the output.

```json
{
  "body_args": {
    "{\"message\": \"Hello World\"}": true
  },
  "body": "{\"message\": \"Hello World\"}"
}
```

## Consume

**HTTP GET**

To consume messages from the `test` topic run:

```sh
curl -H 'Host: kafka.dev' http://localhost:8000/kafka/consumer/http-get
```

if you have `jq` installed you can use the following to pretty print the response:

```sh
curl -H 'Host: kafka.dev' http://localhost:8000/kafka/consumer/http-get -s | jq
```

The response will be a JSON array of messages.

```json
{
  "test": {
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

There is no messages because we didn't produce any yet.

Run the following command to send a message to the `test` topic using kcat:

```sh
echo '{"message": "Hello World"}' | kcat -P -b localhost:9092 -t test
```

Now run again the consume command and you should see the message in the response.

```sh
curl -H 'Host: kafka.dev' http://localhost:8000/kafka/consumer/http-get
```

```json
{
  "test": {
    "partitions": {
      "0": {
        "errcode": 0,
        "high_watermark": 1,
        "records": [
          {
            "key": "",
            "offset": 1,
            "timestamp": 0,
            "value": "{\"message\": \"Hello World\"}"
          }
        ]
      }
    }
  }
}
```

**Server Sent Events**

To consume messages from the `test` topic using Server Sent Events run:

```sh
curl -H 'Host: kafka.dev' http://localhost:8000/kafka/consumer/sse
```

You will get a stream of messages in the response.

```txt
data: {"message": "Hello World"}
topic: test
event: message
id: 0

: keep-alive

: keep-alive

: keep-alive

: keep-alive
```

You can now open another terminal and run the following command to send a message to the `test` topic using kcat:

```sh
echo '{"message": "Hello again"}' | kcat -P -b localhost:9092 -t test
```

You should see the message in the response.

```txt
...

: keep-alive

data: {"message": "Hello again"}
topic: test
event: message
id: 1

: keep-alive
```
