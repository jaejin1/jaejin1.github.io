# Hyperledger Fabric 개발 환경 구성 및 chaincode 배포


Hyperledger Fabric에 대해 알아 보았따. 이제 직접 개발 환경은 올려보자.

<!--more-->

## 개발을 위한 환경

1. docker
2. go
3. [virtualBox](https://www.virtualbox.org/wiki/Downloads)

## Docker VM환경 생성

물론 default환경을 사용해도 좋지만 나중에 여러개의 node를 올려보기 위해서 미리 분리를 하도록 하자.

~~~bash
$ docker-machine create --driver virtualbox blockchain
~~~

다음으로 생성된 VM의 환경변수를 확인하자

~~~bash
$ docker-machine env blockchain
~~~

이 환경을 그대로 적용 시키자

~~~bash
$ eval $(docker-machine env blockchain)
~~~

잘 적용되었는지 확인하려면 ps나 images 명령을 날려보자.

~~~bash
$ docker ps
or
$ docker images
~~~

docker machine을 만들기 전과 결과가 다르다면 성공이다.

## Hyperledger Fabric Docker image받기

~~~bash
docker pull hyperledger/fabric-baseimage:x86_64-0.2.2
docker pull hyperledger/fabric-membersrvc:x86_64-0.6.1-preview
docker pull hyperledger/fabric-peer:x86_64-0.6.1-preview
~~~

baseimage의 경우 TAG를 latest로 해야한다. 

~~~bash
docker tag hyperledger/fabric-baseimage:x86_64-0.2.2 hyperledger/fabric-baseimgae:latest
~~~

~~~bash
docker images
~~~

이제 image들을 받았으니 docker-compose 파일을 하나 작성하자.

~~~yaml
membersrvc:
  image: hyperledger/fabric-membersrvc:x86_64-0.6.1-preview
  ports:
    - "7054:7054"
  command: membersrvc
vp0:
  image: hyperledger/fabric-peer:x86_64-0.6.1-preview
  ports:
    - "7050:7050"
    - "7051:7051"
    - "7053:7053"
  environment:
    - CORE_PEER_ADDRESSAUTODETECT=true
    - CORE_LOGGING_LEVEL=DEBUG
    - CORE_PEER_ID=vp0
    - CORE_PEER_PKI_ECA_PADDR=membersrvc:7054
    - CORE_PEER_PKI_TCA_PADDR=membersrvc:7054
    - CORE_PEER_PKI_TLSCA_PADDR=membersrvc:7054
    - CORE_SECURITY_ENABLED=true
    - CORE_SECURITY_ENROLLID=test_vp0
    - CORE_SECURITY_ENROLLSECRET=MwYpmSRjupbT
  links:
    - membersrvc
  command: sh -c "sleep 5; peer node start --peer-chaincodedev"
~~~

~~~bash
$ docker-compose up
~~~

에러 없이 정상 작동하면 Blockchain server가 실행된 것이다.

## Fabric 소스 준비

chaincode는 go로 되어있는 example을 이용하자.

~~~bash
$ mkdir -p $GOPATH/src/github.com/hyperledger
$ cd $GOPATH/src/github.com/hyperledger
$ git clone https://github.com/hyperledger/fabric.git
$ cd fabric
$ git checkout v0.6.1-preview
~~~

## chaincode 빌드 및 실행

소스를 받았으면 빌드를 한다.

~~~bash
$ cd $GOPATH/src/github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
$ go build
~~~

build되었으면 `chaincode_example02`라는 실행파일이 생성되어 있을 것이다.

### 개발모드 vs 운영모드

개발모드에서는 chaincode가 blockchain 런타임과 별도의 프로세스로 실행된다.

운영모드에서는 chaincode를 deploy하면 blockchain network에 있는 모든 peer들에게 복제가 되고 모든 peer들이 동일한 chaincode로 동작한다. 개발모드는 단일 peer에 chaincode가 붙어서 동작한다.

이제 실제로 블록체인 런타임에 chaincode를 실행하자.

이전에 생성했던 docker-compose.yaml파일을 실행한다.

~~~bash
$ docker-compose up -d 
~~~

chaincode 실행 전 docker VM의 IP를 확인한다.

~~~bash
docker-machine ls
~~~

URL에 `tcp://192.168.99.100:2376` 이라고 (아마 다를 수도 있습니다) 나올 텐데 이를 CORE_PEER_ADDRESS에 넣고 실행시키면된다.

~~~bash
$ cd $GOPATH/src/github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
$ CORE_CHAINCODE_ID_NAME=mycc CORE_PEER_ADDRESS=192.168.99.100:7051 ./chaincode_example02
~~~

요런식으로.

## 테스트

REST API를 통해 테스트 해보자.

### 사용자 등록 및 로그인

먼저 사용자 등록 및 로그인을 해보자. ID, PW는 사전에 정의 되어 있어서 동일 하게 테스트 한다.

~~~http
POST /registrar

{
  "enrollId": "admin",
  "enrollSecret": "Xurw3yU9zI0l"
}
~~~

response로 

~~~json
{
    "OK": "Login successful for user 'admin'."
}
~~~

이라고 응답이 오면 성공한 것이다.

### chaincode deploy

~~~http
POST /chaincode

{
  "jsonrpc": "2.0",
  "method": "deploy",
  "params": {
    "type": 1,
    "chaincodeID":{
        "name": "mycc"
    },
    "ctorMsg": {
        "args":["init", "a", "100", "b", "200"]
    },
    "secureContext": "admin"
  },
  "id": 1
}
~~~

이에 대한 response는 

~~~http
{
    "jsonrpc": "2.0",
    "result": {
        "status": "OK",
        message": "mycc"
    },
    "id": 1
}
~~~

이렇게 돌아오면 성공적이다.

> 만약 개발 모드에서 chaincode를 수정했을시 코드 빌드 -> chaincode 실행 -> 로그인 API호출 -> chaincode deploy 호출 단계로 진행해야 기본 환경 준비가 된다.

### Invoke

~~~http
POST /chaincode

{
  "jsonrpc": "2.0",
  "method": "invoke",
  "params": {
      "type": 1,
      "chaincodeID":{
          "name":"mycc"
      },
      "ctorMsg": {
         "args":["invoke", "a", "b", "10"]
      },
      "secureContext": "admin"
  },
  "id": 3
}
~~~

response

~~~http
{
    "jsonrpc": "2.0",
    "result": {
        "status": "OK",
        "message": "~~~~"
    },
    "id": 3
}
~~~

### Query

invoke가 정상적으로 호출 되었다면 Query로 값을 확인하자.

~~~http
POST /chaincode

{
  "jsonrpc": "2.0",
  "method": "query",
  "params": {
      "type": 1,
      "chaincodeID":{
          "name":"mycc"
      },
      "ctorMsg": {
         "args":["query", "a"]
      },
      "secureContext": "admin"
  },
  "id": 5
}
~~~

response

~~~http
{
    "jsonrpc": "2.0",
    "result": {
        "status": "OK",
        "message": "~~~"
    },
    "id": 3
}
~~~

---

**참고** - [IBM Developer Blog](https://developer.ibm.com/kr/developer-%EA%B8%B0%EC%88%A0-%ED%8F%AC%EB%9F%BC/2017/01/26/hyperledger_fabric_chaincode_develpement_by_devmode/)
