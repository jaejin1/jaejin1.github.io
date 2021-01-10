# Introduction to gRPC


gRPC는 원경의 Client가 Server 단의 함수를 로컬 함수를 호출하듯 부를 수 있게 해준다.  
이때 메시지를 보내고 받는데는 protocol buffers를 사용하고 HTTP/2 기반의 Streaming을 지원하며 REST 대비 빠른 성능을 지원한다.

gRPC의 특징 중 하나는 Java, Ruby, Node, Python, Go등 과 같은 프로그래밍 언어를 지원하고 MSA (마이크로 서비스 아키텍쳐)와 조합이 매력적이다. 

<!--more-->

## Google protocol buffers

구글에서 개발하고 오픈소스로 공개한 직렬화 데이터 구조이다. 다양한 언어를 지원하고 직렬화 속도가 빠르고 파일의 크기가 작다고 한다.

### 사용하기

protocol buffers는 proto file이라는 형태로 정의하는데, protocol buffers는 한가지 언어가 아닌 다양한 언어를 지원하기 때문에 특정언어에 종속성이 없는 형태로 데이터 타입을 정의하는데 이 파일이 proto file이다.

proto file을 컴파일 하면 각 언어에 맞게 클래스 파일을 생성해준다. 예시는 밑의 예제에서 살펴보자.

## 설치

1. golang 1.6 이상
2. install gRPC
    ~~~bash
    $ go get -u google.golang.org/grpc
    ~~~
3. install Protocol Buffers v3
4. install protoc plugin for go
    ~~~bash
    $ go get -u github.com/golang/protobuf/protoc-gen-go
    ~~~
    ~~~bash
    $ export PATH=$PATH:$GOPATH/bin
    ~~~

## Smaple

### 참고 사이트
* https://grpc.io/docs/quickstart/go/

* https://jusths.tistory.com/127

### sample code

* https://github.com/jaejin1/gRPC/tree/master/go

### proto file

#### hi.proto
~~~proto
syntax = "proto3";

package higrpc;

service Hi {
  rpc SayHi (HiRequest) returns (HiReply) {}
  rpc CountHi (HiRequest) returns (HiReply) {}
  rpc JaejinHi (HiRequest) returns (HiReply) {}
}

message HiRequest {
  string name = 1;
}

message HiReply {
  string message = 1;
}
~~~

#### compile hi.proto

~~~bash
$ protoc hi.proto --go_out=plugins=grpc:.
~~~

compile을 하게 되면 hi.pb.go 파일이 생성될 것이다. 이 파일을 server, client 에서 import 시킨다.

### Server

~~~go
package main

import (
	"context"
	"log"
	"net"
	"strconv"

	pb "github.com/jaejin1/grpc/pb"
	"google.golang.org/grpc"
)

const (
	port = ":50051"
)

type server struct{}

func (s *server) SayHi(ctx context.Context, in *pb.HiRequest) (*pb.HiReply, error) {
	log.Printf("Received: %v", in.Name)

	replyMsg := "Hi " + in.Name
	log.Printf("Send back to client: %s", replyMsg)
	return &pb.HiReply{Message: replyMsg}, nil
}

func (s *server) CountHi(ctx context.Context, in *pb.HiRequest) (*pb.HiReply, error) {
	log.Printf("Received: %v, length: %d", in.Name, len(in.Name))

	replyMsg := "Count " + in.Name + " length: " + strconv.Itoa(len(in.Name))
	log.Printf("Send back to client: %s", replyMsg)
	return &pb.HiReply{Message: replyMsg}, nil
}

func (s *server) JaejinHi(ctx context.Context, in *pb.HiRequest) (*pb.HiReply, error) {
	log.Printf("Received: Hi jaejin %v", in.Name)

	replyMsg := "Hi jaejin test"
	log.Printf("Send back to client: %s", replyMsg)
	return &pb.HiReply{Message: replyMsg}, nil
}

func main() {
	log.Printf("grpc server start at port %s", port)

	l, err := net.Listen("tcp", port)
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	s := grpc.NewServer()
	pb.RegisterHiServer(s, &server{})
	if err := s.Serve(l); err != nil {
		log.Fatalf("fail to serve: %v", err)
	}
}
~~~

### client

~~~go
package main

import (
	"context"
	"flag"
	"log"
	"time"

	pb "github.com/jaejin1/grpc/pb"
	"google.golang.org/grpc"
)

const (
	serverAddr  = "localhost:50051"
	defaultName = "world"
)

func main() {
	conn, err := grpc.Dial(serverAddr, grpc.WithInsecure())
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
	c := pb.NewHiClient(conn)

	name := defaultName
	flag.Parse()
	if flag.NArg() > 0 {
		name = flag.Arg(0)
	}

	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()

	r, err := c.SayHi(ctx, &pb.HiRequest{Name: name})
	if err != nil {
		log.Fatalf("could not greet: %v", err)
	}
	log.Printf("Greeting from the Server: %s", r.Message)

	r, err = c.CountHi(ctx, &pb.HiRequest{Name: name})
	if err != nil {
		log.Fatalf("could not count: %v", err)
	}
	log.Printf("Count from the Server: %s", r.Message)

	r, err = c.JaejinHi(ctx, &pb.HiRequest{Name: name})
	if err != nil {
		log.Fatalf("Error !!: %v", err)
	}
	log.Printf("Hi jaejin: %s", r.Message)
}
~~~

## Test

### server 실행
~~~bash
$ go run server/main.go

2019/09/08 15:06:17 grpc server start at port :50051
~~~

### client 실행
~~~bash
$ go run client/main.go

2019/09/08 15:06:44 Greeting from the Server: Hi world
2019/09/08 15:06:44 Count from the Server: Count world length: 5
2019/09/08 15:06:44 Hi jaejin: Hi jaejin test
~~~

### server 확인
~~~bash
$ go run server/main.go

2019/09/08 15:06:17 grpc server start at port :50051
2019/09/08 15:06:44 Received: world
2019/09/08 15:06:44 Send back to client: Hi world
2019/09/08 15:06:44 Received: world, length: 5
2019/09/08 15:06:44 Send back to client: Count world length: 5
2019/09/08 15:06:44 Received: Hi jaejin world
2019/09/08 15:06:44 Send back to client: Hi jaejin test
~~~

다음은 gRPC를 가지고 MSA 특성을 살려보기 위해 go server를 띄워놓고 python client로 접속을 해보려고 한다.
