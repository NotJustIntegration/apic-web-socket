Hey there! In this guide, we'll walk you through setting up a WebSocket connection in IBM API Connect. We'll create an API, implement a WebSocket handler using GatewayScript, and test the connection with a WebSocket client. Let's get started!

## Overview

1. create an API in API Connect for WebSocket requests.
2. Create a GatewayScript to handle the web socket connection.
3. Test the connection using the `websocat` command-line tool.

## Step 1: Creating an API in API Connect

- First, let's create an API in IBM API Connect that'll route WebSocket requests to the right handler.
- In the `paths` section of your API's Swagger file, define a path for the WebSocket connection.
- Don't forget to include the WebSocket schemes in the `schemes` section of the Swagger file. In this guide we have added the `wss` to the API definition.

Example:

```yaml
schemes:
  - https
  - wss
```

## Step 2: Create a GatewayScript to handle the web socket connection

Next, we'll use GatewayScript in API Connect to set up the server-side logic for handling WebSocket connections.
To process the WebSocket handshake, we need to compute the Sec-WebSocket-Accept header value. We'll use the client's Sec-WebSocket-Key header and a predefined GUID for this.

```javascript
var crypto = require('crypto');

function computeSha1Hash(text) {
  var hash = crypto.createHash('sha1');
  hash.update(text);
  return hash.digest('base64');
}


var ws_key=context.get('request.headers.Sec-WebSocket-Key');
const guid = "258EAFA5-E914-47DA-95CA-C5AB0DC85B11";
const ws_accept = ws_key + guid;

var base64AcceptValue = computeSha1Hash(ws_accept);


context.set('message.status.code', 101);
context.set('message.headers.Sec-WebSocket-Accept',base64AcceptValue);
```

## Step 3: Testing the WebSocket Connection
- To test the WebSocket connection, we'll use the websocat command-line tool on our local machine.
- You can use the --insecure flag to bypass SSL certificate validation.
- Provide the WebSocket URL that corresponds to your API Connect instance and the API path you defined earlier.

Example command:

```sh
websocat -v --insecure wss://<gateway endpoint>/<provider organization>/sandbox/websocket-test
```

If everything goes smoothly, you should see the message "Connected to ws".


## Wrapping Up
And that's it! You've successfully set up a WebSocket connection through IBM API Connect. Now you can go ahead and implement the necessary logic for sending and receiving messages using the WebSocket connection, handle errors, and close the connection gracefully when needed. Good luck!

