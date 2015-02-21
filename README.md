# socket.mq
Status: 0.1 DRAFT

HTTP &amp; WebSocket based brokerless message queue
* M2M and inter-process communication
* Coherent and simple architecture
* Determenistic behavior
* Easy-to-use and explicit API
* Language-independent
* Two patterns supported: Request-Reply and Publish-Subscribe
* Both client and server can publish and subcribe events
* Built-in authentication and encryption mechanism
* May be used for Frontend-Backend communication in HTML5 applications
* Frontend can easily communicate with other backend applications at server side
* No restrictions on data format
* Interface versioning built-in
* Proxying possible, eg. all applications proxied at a single http port

## API

Socket.mq implementation may provide Server or Client API, or both

Functions may be implemented synchronously or asynchronously. Depending on the implementation, the functions may have additional arguments or return values in addition to those specified below.

TBD. Should we register other peers with a uniform API for both Server and Client API usage? So that we could refer to applications instead of addresses or keys.

### Server API
| function | description |
| --- | --- |
| listen(port: number, applicationName: string, securityOptions: SecurityOptions, requestCallback: RequestCallback, subscribeCallback: SubscribeCallback) | Starts the server on the given port. |
| close() | Stop the server and close resources |

TBD. listen() method may be quite different when used eg. with Express.

SecurityOptions object contains the following fields (note that fields used to define keys may vary according to the implementation):
* securityEnabled {boolean} True if Server allows Client to establish secure connection to Server
* securityMethod {string or enumeration} Only `OpenPGP` supported by socket.mq version 1.0
* securityRequired {boolean} True if Server accepts only secure connections from Clients
* publicKey {string} Public key of the Server used for authentication and encryption (ASCII-armored format)
* keyRing {string[]} Public keys of the Clients allowed to connect (ASCII-armored format)

RequestCallback function has the following arguments:
* TBD. Is this needed/relevant?? sourceAddress {hostname or IP address} Address where the request came from (TBD. note that it may be the address of the reverse proxy server??)
* request {string} Request id (part of the URL)
* requestBody {binary data} Request body data
* response {ResponseFuction} Function used to response to the request

TBD. ResponseFunction

TBD. SubscribeCallback

## Implementations
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
Socket.mq connection starts when a client connects to a server.
* In Request-Reply pattern, the client makes the request and the server replies. The connection ends after that.
* In Publish-Subscribe pattern, the client connects to the server and both can start listening on each other's events. The connection ends when one of the parties ends it.

### URL Format
TBD

### Request-Reply
Request-Reply pattern uses simple HTTP REST methods.

Request:
* HTTP POST: http://machine:port/socket.mq/application/request-id
* Request body contains the request in application-specific data format.
* TBD. Content-Type field, is it application-specific as well?

Reply:
* Reply body contains the reply in application-specific data format.

TBD. HTTP header telling source address. Nginx reverse proxy -> which HTTP headers telling source address are kept intact as default?

Server responds with the following HTTP status codes:

| status code | used when |
| --- | --- |
| 200 (OK) | Request successful |
| Other error codes | Application-specific errors |

### Publish-Subscribe
TBD. WebSocket

Events are sent only to subscribers to reduce network traffic ie. publisher maintains a list of subscribers.

### Security
![Authentication](http://www.gliffy.com/go/publish/image/7292741/L.png)

OpenPGP is used for both authentication and data encryption. Both client and server need to know each other's public key to enable:
* Client authentication by server (only authorized clients are allowed to connect)
* Server authentication by client (to prevent man-in-the-middle attacks)
* Event-level subscription access control in Publish-Subscribe pattern (at both Client and Server end)
* Request-level access control in Request-Reply pattern
* Data encryption with the public key of the receiver

NOTE: Do not use the built-in security mechanism for Frontend-Backend communication. A private key known by Frontend can be easily hacked. Instead, Frontend authentication should take place at Backend by a per-session or per-user basis.

Request-Reply pattern:

1. Client sends the request body encrypted with the public key of the server and signed with the private key of the client
2. Server decrypts and verifies the request body using its own private key and the public key of the client
3. Server sends the reply body encrypted and signed in a similar fashion
4. Client decrypts and verifies the reply body

HTTP request has the following header field set to indicate Server that the Client wants to establish a secured socket.mq connection: `Security-Method: OpenPGP`. A Server may accept both secure and insecure socket.mq connections from Clients.

Server may respond with the following HTTP error codes:

| status code | used when |
| --- | --- |
| 401 (Unauthorized) | Server fails to decrypt the message, probably due to Client public key not having been registered to the Server |
| 403 (Forbidden) | Authenticated Client is not authorized to make the request |
| 496 (No Cert) | Client request did not have Security-Method field and Server does not accept insecure connections |

Publish-Subscribe pattern:

TBD

### Public Key Interface
An application acting as a socket.mq server may offer its public key at http://machine/socket.mq/public-key. Public key request does not require authentication. The Public Key Interface serves the following purposes:
1. An administrator can manually download and install server public key to the client's key repository at system commissioning time.
2. When configured so, Client can automatically download and install the Server public key when it it connects to the Server at the first time. After that, Client should not automatically accept a changed public key without manual approval to prevent man-in-the-middle attacks.

### Interface Versioing
TBD
Should the version number be part of the url? Or as HTTP header?

### Data Format
TBD


## Comparison with Other MQ and Similar Solutions

### socket.io
Socket.mq resembles [socket.io](http://socket.io) in many ways. Socket.io can be  replaced by socket.mq with a little effort. Major differences include:
* Socket.io is intended for Frontend-Backend communication only whereas socket.io allows communication between multiple parties in a cross-platform manner
* Socket.io has the ability to downgrade from WebSocket to HTTP polling if WebSocket transport fails
* Socket.mq architecture guarantees more determenistic behavior than socket.io in particular concerning transport times and reliability
* Socket.mq JavaScript client-side library is installed separately (as a bower package) whereas socket.io bundles together the backend and client-side libraries

### ZeroMQ
* ZeroMQ has a more feature-rich functionality and API
* ZeroMQ is a message queue library whereas socket.mq is primarily a message queue architecture
* Each ZeroMQ endpoint listens to its own dedicated TCP port. You cannot proxy them to a single TCP port for M2M communication.
* ZeroMQ performs better while socket.mq has quite a good performance as well when considering long-term Publisher-Subscriber connections.
* ZeroMQ has no support for authentication and encryption
* ZeroMQ has no built-in interface versioning

## Roadmap

### Version 1
Currently at draft status

### Version 2
Support for service discovery

## References

* OpenPGP [standard](https://tools.ietf.org/html/rfc4880) and an [overview](http://www.spywarewarrior.com/uiuc/gpg/gpg-com-4.htm) of GnuPG commands that explains the principles of PGP as well

