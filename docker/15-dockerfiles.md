# Dockerfiles
Dockerfiles are the source code for docker *images*.

```dockerfile
FROM sourcerepo:tag

WORKDIR ...
COPY ... ...
RUN command 1 && \
  command 2

ENTRYPOINT ...
CMD ...
```

## Basic Dockerfile
server.js
```javascript
var http = require('http');
var handleRequest = function(request, response) {
  response.writeHead(200);
  response.end('Hello World!');
};
var helloServer = http.createServer(handleRequest);
helloServer.listen(8080);
```

```dockerfile
FROM node
EXPOSE 8080
COPY server.js .
CMD node server.js
```


### Docker multi-stage builds

#### Static client-side app example
```dockerfile
FROM node:alpine as builder
WORKDIR /code
ADD package.json .
ADD package-lock.json .
RUN npm install
ADD . /code
RUN npm run build

FROM nginx:alpine
EXPOSE 80
COPY --from=builder /code/dist/ /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
ADD start.sh /usr/local/bin
CMD /usr/local/bin/start.sh
```
We don't want to include all of the node/npm dependencies used to compile the static assets in the 
final production shipped image.


#### Creating dev-env / testing images
```dockerfile
FROM golang:alpine AS stage0
…

FROM scratch AS release
COPY --from=stage0 /binary0 /bin
COPY --from=stage1 /binary1 /bin

FROM golang:alpine AS dev-env
COPY --from=release / /
ENTRYPOINT ["ash"]

FROM golang:alpine AS test
COPY --from=release / /
RUN go test …

# Default when --target not specified
FROM release
```
