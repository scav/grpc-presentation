---
title: Marp slide deck
description: An example slide deck created by Marp CLI
author: Yuki Hattori
keywords: marp,marp-cli,slide
url: https://marp.app/
image: https://marp.app/og-image.jpg
theme: gaia
marp: true
---
<!-- _class: lead -->
# gRPC

`fast simple rpc`

---

# Summary

1. Connects service as RPC
2. Protocol Buffers
3. Java, **`Rust`**, TypeScript, Go
4. HTTP/2 transport ++

---

# Overview

![](./../img/landing-2.svg)

---

# Protocol Buffers

A very simple, but powerful binary serialization 

```protobuf
service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

---

# Protocol Buffers

Unary client request

```protobuf
service HelloService {
  rpc SayHello(HelloRequest) returns (HelloResponse);
}
```

Client sends a request and gets a stream back
```protobuf
service HelloService {
  rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse);
}
```

---

# Protocol Buffers

Client sends a stream and get a response back
```protobuf
service HelloService {
  rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);
}
```

Client and server send streams back and forth independently
```protobuf
service HelloService {
  rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);
}
```

---

# Server


```javascript
function sayHello(call, callback) {
  callback(null, {message: 'Hello ' + call.request.name});
}

function sayHelloAgain(call, callback) {
  callback(null, {message: 'Hello again, ' + call.request.name});
}

function main() {
  var server = new grpc.Server();
  server.addService(hello_proto.Greeter.service,
                         {sayHello: sayHello, sayHelloAgain: sayHelloAgain});
  server.bindAsync('0.0.0.0:50051', grpc.ServerCredentials.createInsecure(), () => {
    server.start();
  });
}
```

---

# Client

```javascript
function main() {
  var client = new hello_proto.Greeter('localhost:50051',
                                       grpc.credentials.createInsecure());
  client.sayHello({name: 'you'}, function(err, response) {
    console.log('Greeting:', response.message);
  });
  client.sayHelloAgain({name: 'you'}, function(err, response) {
    console.log('Greeting:', response.message);
  });
}
```