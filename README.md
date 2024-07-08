# MainAPIService

This service accepts client API requests, processes them, and sends the data to a message queue. The worker node processes the data and sends the response back to the client.

## Environment Variables

1. `PORT`: The port on which the service listens, default is 8080.
2. `PULSAR_URL`: The URL to connect to Pulsar.
3. `PULSAR_TOKEN`: The token to connect to Pulsar.
4. `TIMEOUT`: The client timeout duration, default is 5 minutes. The format should follow the specification at https://pkg.go.dev/time#ParseDuration.
5. `DEBUG`: When set, the service runs in debug mode.

## Build and Run

To compile the service:

```bash
go build
```

To run the service directly:

```bash
go run .
```

## API Specification

### Message Queue Data

Data is sent to the topic `"model-${modelName}"`, for example, `model-gpt-3.5-turbo`.

The data is in JSON format and has the following structure:

```json
{
  "request_id": string,
  "stream": boolean,
  "endpoint": string,
  "data": {
    // The body of the original request remains unchanged
    // Refer to https://platform.openai.com/docs/api-reference/chat for format details
  }
}
```

### Example

#### stream=true

```json
{
  "request_id": "4e4a670a-b66a-481f-a37a-9b77af6c047e",
  "stream": true,
  "endpoint": "http://api-01.api.default.svc.cluster.local/res/ws",
  "data": {
    "model": "gpt-4",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {
        "role": "user",
        "content": "Hello!"
      }
    ],
    "stream": true
  }
}
```

#### stream=false

```json
{
  "request_id": "4e4a670a-b66a-481f-a37a-9b77af6c047e",
  "stream": false,
  "endpoint": "http://api-01.api.default.svc.cluster.local/res",
  "data": {
    "model": "gpt-4",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {
        "role": "user",
        "content": "Hello!"
      }
    ]
  }
}
```

The values of `.stream` and `.data.stream` are generally the same (if `.data.stream` is not set, `.stream` is `false`). When `true`, the worker node must send data in real-time through WebSocket. Even if the upstream API/local model does not support streaming, the worker node should still send responses in WebSocket and streaming format. The timing for establishing the WebSocket can be flexible (e.g., if streaming is not supported but requested, the WebSocket can be established after the response is complete to send the result).

## Response Format

### stream=true

**Request Not Completed**

```json
{
  "request_id": string,
  "end": false,
  "data": {
    // The body of the original response fragment remains unchanged
  }
}
```

**Request Completed**

```json
{
  "request_id": string,
  "end": true,
  "data": {
    // The body of the original response fragment remains unchanged
  }
}
```

### stream=false

```json
{
  "request_id": string,
  "data": {
    // The body of the original response remains unchanged
  }
}
```