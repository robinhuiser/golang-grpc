# Quick start guide

My notes and some updates following the [Go Quick Start](https://grpc.io/docs/quickstart/go/).

## Setup local env

~~~bash
# Install the module
$ export GO111MODULE=on # Enable module-aware mode
$ go get google.golang.org/grpc@v1.28.1

# Install the protobuf CLI
$ brew install protobuf
$ protoc --version
libprotoc 3.11.4

# Install the protoc plugin for go
$ go get github.com/golang/protobuf/protoc-gen-go
~~~

## Copy example

~~~bash
# Make a copy of the example helloworld code
$ mkdir quickstart && \
   cp -R $GOPATH/src/google.golang.org/grpc/examples/helloworld ./quickstart

# Setup the example as a module
$ cd quickstart/helloworld && \
   go mod init quickstart/helloworld

# Adjust import paths to reference local packages
$ perl -pi -e 's|google.golang.org/grpc/||g' greeter_{client,server}/main.go
~~~

Update greeter_server/main.go the go path (is now quickstart, not examples)

## Run example

From the `./quickstart/helloworld` directory:

~~~bash
# Terminal #1
$ go run greeter_server/main.go

# Terminal #2
$ go run greeter_client/main.go
2020/05/10 18:35:52 Greeting: Hello world

# Terminal #1 now shows:
2020/05/10 18:35:52 Received: world
~~~

## Modify the example

I'll update the application with an extra server method (`SayHelloAgain`)- updated `./quickstart/helloworld/helloworld.proto`:

~~~go
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}

  // My custom method: Send another greeting
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}
~~~

Next, recompile the updated proto file:

~~~bash
# Run this command from the quickstart/helloworld dir
$ protoc -I helloworld/ helloworld/helloworld.proto  \
   --go_out=plugins=grpc:helloworld

# The protocol buffer defs are now updated:
l$ s -ltr helloworld/helloworld.pb.go
-rw-r--r--  1 robin  staff  12384 10 May 18:49 helloworld/helloworld.pb.go
~~~

Now, we need to implement the method in the server and client.

### SayHelloAgain server impl

Add the the file `greeter_server/main.go` the function:

~~~go
// My SayHelloAgain server extension
func (s *server) SayHelloAgain(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
    log.Printf("Received: %v", in.GetName())
    return &pb.HelloReply{Message: "Hello again " + in.GetName()}, nil
}
~~~

### SayHelloAgain client impl

Add the the file `greeter_server/main.go` the function:

~~~go
// My SayHelloAgain client extension
r, err = c.SayHelloAgain(ctx, &pb.HelloRequest{Name: name})
if err != nil {
    log.Fatalf("could not greet: %v", err)
}
log.Printf("Another greeting: %s", r.GetMessage())
~~~

Run both the server and client again from different terminals... and now the client returns:

~~~bash
$ go run greeter_client/main.go
2020/05/10 18:57:08 Greeting: Hello world
2020/05/10 18:57:08 Another greeting: Hello again world
~~~

That was easy.
