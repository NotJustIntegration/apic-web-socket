<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->

- [API Connect WebSocket Integration](#api-connect-websocket-integration)
  * [Approach 1: WebSocket Upgrade Policy](#approach-1-websocket-upgrade-policy)
      - [1. On the server, create a new directory and Install the dependencies for the Node.js WebSocket server:](#1-on-the-server-create-a-new-directory-and-install-the-dependencies-for-the-nodejs-websocket-server)
      - [2. creat a file in your server, name it and copy the following nodejs code:](#2-creat-a-file-in-your-server-name-it-and-copy-the-following-nodejs-code)
      - [3. Start the Node.js WebSocket server:](#3-start-the-nodejs-websocket-server)
      - [4. In API Connect, create a new API with a WebSocket Upgrade policy.](#4-in-api-connect-create-a-new-api-with-a-websocket-upgrade-policy)
      - [5. Set the target URL of the WebSocket Upgrade policy to the Node.js WebSocket server (in this example we set the port to 8091).](#5-set-the-target-url-of-the-websocket-upgrade-policy-to-the-nodejs-websocket-server-in-this-example-we-set-the-port-to-8091)
      - [6. Include the wss scheme in the definition file of the API. Example:](#6-include-the-wss-scheme-in-the-definition-file-of-the-api-example)
      - [7. Deploy the API to the API Connect gateway.](#7-deploy-the-api-to-the-api-connect-gateway)
      - [8. Test the WebSocket connection using the websocat command:](#8-test-the-websocket-connection-using-the-websocat-command)
  * [Approach 2: API Connect GatewayScript](#approach-2-api-connect-gatewayscript)
  * [Overview](#overview)
      - [1. Creating an API in API Connect](#1-creating-an-api-in-api-connect)
      - [2. Create a GatewayScript to handle the web socket connection](#2-create-a-gatewayscript-to-handle-the-web-socket-connection)
      - [3. Testing the WebSocket Connection](#3-testing-the-websocket-connection)
  * [Wrapping Up](#wrapping-up)

<!-- TOC end -->

<!-- TOC --><a name="api-connect-websocket-integration"></a>
# API Connect WebSocket Integration

This repository demonstrates two different approaches for integrating WebSockets in API Connect.

<!-- TOC --><a name="approach-1-websocket-upgrade-policy"></a>
## Approach 1: WebSocket Upgrade Policy

In this approach, API Connect uses a WebSocket Upgrade policy to handle WebSocket connections.In this case, the gateway will act as a passthrough web socket gateway

<!-- TOC --><a name="1-on-the-server-create-a-new-directory-and-install-the-dependencies-for-the-nodejs-websocket-server"></a>
#### 1. On the server, create a new directory and Install the dependencies for the Node.js WebSocket server:

```bash
npm install ws
```

<!-- TOC --><a name="2-creat-a-file-in-your-server-name-it-and-copy-the-following-nodejs-code"></a>
#### 2. creat a file in your server, name it and copy the following nodejs code:
```javascript
const WebSocket = require('ws');

const server = new WebSocket.Server({ port: 8091 });

server.on('connection', (socket) => {
  console.log('Client connected');

  // Send a welcome message to the client
  socket.send('Welcome to the WebSocket server!');

  // Listen for messages from the client
  socket.on('message', (message) => {
    console.log(`Received message: ${message}`);

    // Convert the message to a string and then to uppercase
    const processedMessage = `Processed message: ${message.toString().toUpperCase()}`;

    // Send the processed message back to the client
    socket.send(processedMessage);
  });

  // Handle disconnections
  socket.on('close', () => {
    console.log('Client disconnected');
  });
});
```
<!-- TOC --><a name="3-start-the-nodejs-websocket-server"></a>
#### 3. Start the Node.js WebSocket server:

```bash
node websocket_server.js
```
<!-- TOC --><a name="4-in-api-connect-create-a-new-api-with-a-websocket-upgrade-policy"></a>
#### 4. In API Connect, create a new API with a WebSocket Upgrade policy.
<!-- TOC --><a name="5-set-the-target-url-of-the-websocket-upgrade-policy-to-the-nodejs-websocket-server-in-this-example-we-set-the-port-to-8091"></a>
#### 5. Set the target URL of the WebSocket Upgrade policy to the Node.js WebSocket server (in this example we set the port to 8091).
Example: 
```yaml
target-url: http://<server ip address>:8091
```

<!-- TOC --><a name="6-include-the-wss-scheme-in-the-definition-file-of-the-api-example"></a>
#### 6. Include the wss scheme in the definition file of the API. Example:

```yaml
schemes:
  - https
  - wss
```
<!-- TOC --><a name="7-deploy-the-api-to-the-api-connect-gateway"></a>
#### 7. Deploy the API to the API Connect gateway.
<!-- TOC --><a name="8-test-the-websocket-connection-using-the-websocat-command"></a>
#### 8. Test the WebSocket connection using the websocat command:

```bash
websocat -v --insecure wss://your-api-connect-gateway/your-websocket-path
```

<!-- TOC --><a name="approach-2-api-connect-gatewayscript"></a>
## Approach 2: API Connect GatewayScript
In this approach, we'll walk you through setting up a WebSocket connection in IBM API Connect. We'll create an API, implement a WebSocket handler using GatewayScript, and test the connection with a WebSocket client. Let's get started!

<!-- TOC --><a name="overview"></a>
## Overview

1. create an API in API Connect for WebSocket requests.
2. Create a GatewayScript to handle the web socket connection.
3. Test the connection using the `websocat` command-line tool.

<!-- TOC --><a name="1-creating-an-api-in-api-connect"></a>
#### 1. Creating an API in API Connect

- First, let's create an API in IBM API Connect that'll route WebSocket requests to the right handler.
- In the `paths` section of your API's Swagger file, define a path for the WebSocket connection.
- Don't forget to include the WebSocket schemes in the `schemes` section of the Swagger file. In this guide we have added the `wss` to the API definition.

Example:

```yaml
schemes:
  - https
  - wss
```

<!-- TOC --><a name="2-create-a-gatewayscript-to-handle-the-web-socket-connection"></a>
#### 2. Create a GatewayScript to handle the web socket connection

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

<!-- TOC --><a name="3-testing-the-websocket-connection"></a>
#### 3. Testing the WebSocket Connection
- To test the WebSocket connection, we'll use the websocat command-line tool on our local machine.
- You can use the --insecure flag to bypass SSL certificate validation.
- Provide the WebSocket URL that corresponds to your API Connect instance and the API path you defined earlier.

Example command:

```sh
websocat -v --insecure wss://<gateway endpoint>/<provider organization>/sandbox/websocket-test
```

If everything goes smoothly, you should see the message "Connected to ws".


<!-- TOC --><a name="wrapping-up"></a>
## Wrapping Up
And that's it! You've successfully set up a WebSocket connection through IBM API Connect. Now you can go ahead and implement the necessary logic for sending and receiving messages using the WebSocket connection, handle errors, and close the connection gracefully when needed. Good luck!


