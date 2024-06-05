# Building microservices in Go and Python using gRPC


[이전글](https://jaejin1.github.io/grpc1/)에서 작성 한 go server를 가지고 python 에서 client로 접속해보자.

<!--more-->

## 코드

### server

이전글에서 작성한 go server 그대로 이용한다.
[https://github.com/jaejin1/gRPC/blob/master/go/server/main.go](https://github.com/jaejin1/gRPC/blob/master/go/server/main.go)

### client

client이기 때문에 간단하게 python으로 작성해보자.

먼저 [proto file](https://github.com/jaejin1/gRPC/blob/master/python/client/hi.proto)을 python 코드로 변환 한다. 물론 전에 사용한 [proto file](https://github.com/jaejin1/gRPC/blob/master/python/client/hi.proto)을 그대로 사용한다.

~~~bash
$ python -m grpc_tools.protoc -I./ --python_out=. --grpc_python_out=. ./hi.proto
~~~

proto file을 compile하게 되면 `hi_pb2_grpc.py`, `hi_pb2.py` 2개의 파일이 생성된다.
이 2파일을 다음 작성할 python코드에 import 시킨 뒤 사용한다.

`main.py`

~~~python
"""The Python implementation of the gRPC route guide client."""

from __future__ import print_function

import random
import logging

import grpc

import hi_pb2
import hi_pb2_grpc

def run():
    with grpc.insecure_channel('localhost:50051') as channel:
        stub = hi_pb2_grpc.HiStub(channel)

        ## JaejinHi
        response = stub.JaejinHi(hi_pb2.HiRequest(name='test test test test !!'))
    print("Greeter client received: " + response.message)

if __name__ == '__main__':
    logging.basicConfig()
    run()
~~~

## 실행

먼저 Server를 실행한다.

~~~bash
$ go run main.go

2019/09/08 22:51:48 grpc server start at port :50051
~~~

python으로 작성한 client를 실행해보자.

~~~bash
$ python main.py

Greeter client received: Hi jaejin test
~~~

다시 server를 확인하면 데이터가 온 것을 볼 수 있다.

~~~bash
$ go run main.go

2019/09/08 22:51:48 grpc server start at port :50051
2019/09/08 22:52:22 Received: Hi jaejin test test test test !!
2019/09/08 22:52:22 Send back to client: Hi jaejin test
~~~
