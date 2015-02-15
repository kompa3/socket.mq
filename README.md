# socket.mq
HTTP &amp; WebSocket based brokerless message queue
* M2M and inter-process communication
* Simple architecture
* Simple API
* Language-independent
* Two patterns supported: Request-Reply and Publish-Subscribe
* Both client and server can publish and subcribe events
* Built-in authentication mechanism
* Encryption possible with SSL when needed
* May be used for Frontend-Backend communication in HTML5 applications
* Frontend can easily communicate with other backend applications at server side
* JSON and binary data supported
* Interface versioning built-in
* Nginx-friendly
* Especially handy in such network environments where only http traffic is allowed

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

## Communication Architecture Overview

### Request-Reply
REST

### Publish-Subscribe
WebSocket

### Authentication
TBD

Do not use the built-in authentication mechanism with Frontend. Any key known by Frontend is automatically accessible by anyone. Instead, Frontend authentication should take place at Backend with a per-session or per-user manner.

### Interface Versioing
TBD
Should the version number be part of the url?

### Data Format
TBD
Google Protocols for binary data?

## Comparison with Other MQ and Similar Solutions

### socket.io
Socket.mq resembles [socket.io](http://socket.io) in many ways. Socket.io can be  replaced by socket.mq with a little effort. Major differences include:
* Socket.io is fixed to JavaScript language
* Socket.io downgrades from WebSocket to HTTP polling if WebSocket transport has been prevented by network configuration
* Socket.mq client-side library must be installed separately (as a bower package)

### ZeroMQ
* ZeroMQ has a more complex functionality and API
* ZeroMQ is a message queue library whereas socket.mq is primarily a communication architecture. While socket.mq architecture is so simple you don't necessarily need a library to use it.
* Each ZeroMQ endpoint listens to its own dedicated TCP port. You cannot reverse-proxy them to a single TCP port for M2M communication.
* ZeroMQ performs better while socket.mq has quite a good performance as well when considering long-term Publisher-Subscriber connections.