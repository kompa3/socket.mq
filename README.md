# socket.mq
HTTP &amp; WebSocket based brokerless message queue for enterprise solutions
* M2M and inter-process communication
* Simple architecture
* Simple API
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
