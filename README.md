# socket.mq
Status: DRAFT

HTTP &amp; WebSocket based brokerless message queue
* M2M and inter-process communication
* Simple architecture
* Easy-to-use API
* Language-independent
* Two patterns supported: Request-Reply and Publish-Subscribe
* Both client and server can publish and subcribe events
* Built-in authentication and encryption mechanism
* May be used for Frontend-Backend communication in HTML5 applications
* Frontend can easily communicate with other backend applications at server side
* No restrictions on data format
* Interface versioning built-in
* Proxying possible, eg. all applications proxied at a single http port

## API Examples
TBD

## Network Architecture Examples
### Brokerless Communication
![Example](http://www.gliffy.com/go/publish/image/7237141/L.png)

Each application that acts as a server listens to its own port. TCP ports can be defined freely. An application that acts only as a client does not listen to any port.

### M2M Communication Restricted to Port 80
![Example](http://www.gliffy.com/go/publish/image/7236917/L.png)

Some network environments may restrict traffic between machines to http port 80. HTTP endpoint (eg. [Nginx](http://nginx.org)) acts as a reverse proxy for multiple applications.

### Frontend Communicates with Other Backend Applications
![Example](http://www.gliffy.com/go/publish/image/7237101/L.png)

Nodejs Backend acts as a HTTP proxy (eg. with [node-http-proxy](https://github.com/nodejitsu/node-http-proxy)) to allow Frontend to communicate with other applications at server side.

## Architecture Overview

### Client-Server Architecture
Socket.mq connection is established by the cli

In Request-Reply pattern, client makes the request and server replies.

In Publish-Subscribe pattern, 

### Request-Reply
Request-Reply pattern uses simple HTTP REST methods.
#### Request
HTTP POST: http://machine:port/socket.mq/application/request-id


Request body contains the request in application-specific data format.
TBD. Content-Type field, is it application-specific as well?

#### Reply
HTTP/1.1 200 OK (or any of the error codes)

Reply body contains the reply in application-specific data format.

### Publish-Subscribe
WebSocket

### Authentication and Encryption
OpenPGP is used for both client authentication and data encryption. Both client
1. aer
2. 

TBD
Do not use the built-in authentication mechanism with Frontend. Any credentials known by Frontend are automatically accessible by anyone. Instead, Frontend authentication should take place at Backend by a per-session or per-user basis.

### Interface Versioing
TBD
Should the version number be part of the url? Or as HTTP header?

### Data Format
TBD
Google Protocols for binary data?

## Comparison with Other MQ and Similar Solutions

### socket.io
Socket.mq resembles [socket.io](http://socket.io) in many ways. Socket.io can be  replaced by socket.mq with a little effort. Major differences include:
* Socket.io is fixed to JavaScript language
* Socket.io has the ability to downgrade from WebSocket to HTTP polling if WebSocket transport fails
* Socket.mq client-side library is installed separately (as a bower package) whereas in socket.io the backend and client-side libraries are bundled together

### ZeroMQ
* ZeroMQ has a more feature-rich functionality and API
* ZeroMQ is a message queue library whereas socket.mq is primarily a message queue architecture
* Each ZeroMQ endpoint listens to its own dedicated TCP port. You cannot proxy them to a single TCP port for M2M communication.
* ZeroMQ performs better while socket.mq has quite a good performance as well when considering long-term Publisher-Subscriber connections.
* ZeroMQ has no support for authentication and encryption

## Roadmap

### Version 1
Currently at draft status

### Version 2
Support for service discovery

### Version 3
Certificate support so that server public keys do not need to be configured for each client manually.
