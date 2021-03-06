# socket.mq
Status: 0.1 DRAFT / IN PROGRESS

TBD: Check AMQP over WebSocket http://docs.oasis-open.org/amqp-bindmap/amqp-wsb/v1.0/amqp-wsb-v1.0.html

HTTP &amp; WebSocket based brokerless message queue
* M2M, inter-process, and Frontend-Backend communication
* Coherent and simple architecture that guarantees determenistic behavior
* Easy-to-use explicit API
* Language-independent cross-platform communication
* Two patterns supported: Request-Reply and Publish-Subscribe
* Built-in security (authentication and encryption)
* Built-in interface versioning

## API

Socket.mq implementation may provide Server or Client API, or both

Functions may be implemented synchronously or asynchronously. Depending on the implementation, the functions may have additional arguments or return values in addition to those specified below.

### Event Interface
| member | arguments | description |
| --- | --- | --- |
| publish() | event: string, data: binary | Publish the event to all the subscribers. Data (optional) is in application-specific format. |
| subscribe() | applicationID: string, event: string | Subscribe to the event of a connected application. |
| subscribe callback | See below | Callback called when an application wants to subscribe to my event. Optional. If not defined all events can be subscribed to by connected applications. |
| event callback | See below | Callback called when a subscribed event has been received from the connected publisher. |

Subscribe callback has the following arguments:
* event {string} Event subscribed to
* applicationID {string} ID of application that wants to subscribe
* accept() {function} Accept event subscription by calling this function
* reject(errorCode: number) {function} Reject event subscription by calling this fuction. The error code is returned as HTTP status code.

Event callback has the following arguments:
* applicationID {string} Application ID that published the event
* event {string} Event ID
* data {binary data} Event data in application-specific format

### Server Interface
| member | arguments | description |
| --- | --- | --- |
| constructor or listen() method | port: number, applicationName: string, securityOptions: SecurityOptions | Starts the server on the given port. |
| request callback | See below | Callback called when Client makes a request to Server |
| connect callback | See below, TBD | TBD, is this rational? |
| destructor or close() method | Stop the server and free resources |

TBD. listen() method may be quite different when used eg. with Express.

SecurityOptions contains the following fields (note that fields used to define keys may vary according to the implementation):
* securityEnabled {boolean} True if Server allows Clients to establish secure connection to Server
* securityMethod {string or enumeration} Only `OpenPGP` supported by socket.mq version 1.0
* securityRequired {boolean} True if Server accepts only secure connections from Clients
* publicKey {string} Armored public key of the Server used for authentication and encryption
* keyRing {string[]} Armored public keys of the Clients allowed to connect

Note that the application identifier is encoded as part of the public key.

Request callback has the following arguments:
* request {string} Request id (part of the URL)
* data {binary data} Request body data in application-specific format
* clientID {string} Client application ID
* respond() {see below} Function used to respond to the request

Respond function has the following arguments:
* respondStatus {number} HTTP status code returned to Client
* data {binary data} Respond body data in application-specific format, optional

## Implementations
* [Socket.mq-nodejs](https://github.com/kompa3/socket.mq-nodejs)

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

### Mobile Application

TBD. Mobile application generates its own key send posts the public key via request interface, providing application-specific (temporary?) credentials for "logging in". After that, no logging-in needed since authentication by keys.

## Architecture Overview

### Connection
Socket.mq connection parties are called applications. A connection starts when Client connects to Server.
* In Request-Reply pattern, the Client makes the request and the Server replies. The connection ends after that.
* In Publish-Subscribe pattern, the Client connects to the Server and both can start listening on each other's events. The connection ends when one of the parties ends it.

Connection can be either plain or secure connection.
* In plain connection, parties are not authenticated securely and the traffic is in plain non-encrypted format.
* In secure connection, parties are authenticated securely and the traffic is encrypted.

### Request-Reply

TBD. What about changing HTTP REST to WebSocket, ie. Request-Reply happens over an established WebSocket communciation channel?
* Improved performance
* More logical API (connect event)
* Server can request from Client

Request-Reply pattern uses simple HTTP REST methods.

Request:
* HTTP POST: http://machine:port/socket.mq/app1/request-id where
** machine is the address (host name or IP address)
** port is TCP port listened to by the Server application
** app1 is the application ID of the Server connected to
** request-id is the identifier of the request made
* Request body contains the request data in application-specific format
* HTTP header fields:
** Client-ID: Client application ID for plain connections
** TBD. Content-Type field, is it application-specific?
** TBD. Interface version

Reply:
* HTTP status code, see below
* Reply body contains the reply data in application-specific format

Server responds with the following HTTP status codes:

| status code | used when |
| --- | --- |
| 200 (OK) | Request successful |
| Other error codes | Application-specific errors |

### Publish-Subscribe
Publish-Subscribe pattern uses WebSocket protocol to establish a live connection between the applications.

Connection by Client:
* WebSocket URL: ws://machine:port/socket.mq/app1
* 
TBD. WebSocket

subprotocol: socket.mq

Events are sent only to subscribers to reduce network traffic. A publisher maintains a list of event subscribers.

### Security
![Authentication](http://www.gliffy.com/go/publish/image/7292741/L.png)

OpenPGP is used for both authentication and data encryption. Both client and server need to know each other's public key to enable:
* Client authentication by server (only authorized clients are allowed to connect)
* Server authentication by client (to prevent man-in-the-middle attacks)
* Event-level subscription access control in Publish-Subscribe pattern (at both Client and Server end)
* Request-level access control in Request-Reply pattern
* Data encryption with the public key of the receiver

Rationale for using PGP instead of SSL or TSL:
* Socket.mq is intended for m2m communication where there might not exist Internet connection, CA servers, or DNS names.
* See a related [article](http://www.techrepublic.com/blog/it-security/why-not-use-openpgp-for-web-authentication/) about using OpenPGP instead of TLS for web authentication

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

* An administrator can manually download and install server public key to the client's key repository at system commissioning time.
* When configured so, Client can automatically download and install the Server public key when it it connects to the Server at the first time. After that, Client should not automatically accept a changed public key without manual approval to prevent man-in-the-middle attacks.

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
Currently in progress

### Version 2
Support for service discovery

## Resources

* OpenPGP [standard](https://tools.ietf.org/html/rfc4880) and an [overview](http://www.spywarewarrior.com/uiuc/gpg/gpg-com-4.htm) of GnuPG commands that explains the principles of PGP as well

