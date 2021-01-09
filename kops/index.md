# Kubernetes cluster on aws with kops


MacOS에서 cluster를 구성한뒤 AWS에 노드들을 띄우는 작업을 해봅시다.

<!--more-->

## kubernetes cluster setting for mac

### 설치

먼저 
kubernetes를 다루기 위해 kubectl을 다운로드 한다.

~~~bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl
~~~

에러가 날 것인데 최신 릴리즈 버전을 확인하자

~~~bash
curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt
~~~

예를들어 위의 결과가 이렇게 나왔다면.

~~~bash
v1.12.2

curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.12.2/bin/darwin/amd64/kubectl
~~~

이런식으로 다운로드 받으면 된다.

~~~bash
chmod +x ./kubectl
~~~

실행 모드로 변경하고 

~~~bash
sudo mv ./kubectl /usr/local/bin/kubectl
~~~

Move the binary in to your PATH

### 설치 확인

~~~bash
kubectl version

>
Client Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.2", GitCommit:"17c77c7898218073f14c8d573582e8d2313dc740", GitTreeState:"clean", BuildDate:"2018-10-24T06:54:59Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.6", GitCommit:"a21fdbd78dde8f5447f5f6c331f7eb6f80bd684e", GitTreeState:"clean", BuildDate:"2018-07-26T10:04:08Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
~~~

이런식으로 client와 server가 둘다 나와야 잘 설치 된 것이다.

### AWS CLI

AWS에 노드를 띄우기 위해 AWS의 설정을 해준다.

~~~bash
$ sudo apt update
$ sudo apt install -y awscli
$ aws configure

>
public, secret key 입력
~~~

### Kops

쉽게 클러스터를 전개 하기 위해 kops를 설치한다.

~~~bash
$ wget https://github.com/kubernetes/kops/releases/download/1.10.0/kops-darwin-amd64
$ chmod +x kops-darwin-amd64
$ sudo mv kops-darwin-amd64 /usr/local/bin/kops
~~~

설치 완료되면 

~~~bash
$ kops 
~~~

명령어가 잘 먹을 것이다.

kops로 전개한 AWS EC2에 접속하기 위한 key 생성을 해보자

### Key

~~~bash
$ ssh-keygen -q -f ~/.ssh/id_rsa -N ""
~~~

or

~~~bash
$ kops create secret --name awskrug.k8s.local sshpublickey admin -i ~/.ssh/id_rsa.pub
~~~

이제 본격적인 Cluster를 만들어보자.

### Cluster

~~~bash
$ export KOPS_CLUSTER_NAME=awskrug.k8s.local
$ export KOPS_STATE_STORE=s3://kops-awskrug-test
$ aws s3 mb ${KOPS_STATE_STORE} --region us-east-1
~~~

환경 변수 설정을 해주고 S3에 데이터를 저장하기위해 버킷을 생성을 한다.

~~~bash
kops create cluster \
--cloud=aws \
--name=${KOPS_CLUSTER_NAME} \
--state=${KOPS_STATE_STORE} \
--master-size=t2.medium \
--node-size=t2.medium \
--node-count=2 \
--zones=us-east-1b,us-east-1b \
--ssh-public-key=~/.ssh/k8s.pub \
--network-cidr=10.10.0.0/16

~~~

원하는 대로 설정이 가능하고  `--ssh-public-key=~/.ssh/k8s.pub` 부분은 위에 Key 생성한 걸로 해주면 노드에 접속할 수 있다.

노드 접속은

~~~bash
ssh -i ~/.ssh/k8s admin@184.73.133.76
~~~

이런식으로 EC2 instance의 IP를 써주면 접속이 가능하다.

### Update Cluster

- 클러스터 생성 하기 전 수정하기

~~~bash
$ kops edit cluster --name=${KOPS_CLUSTER_NAME}
~~~

### Create Cluster

- kops update에 --yes 옵션을 주면 실제 클러스터가 생성된다.

~~~bash
$ kops update cluster --name=${KOPS_CLUSTER_NAME} --yes
~~~

### Validate Cluster

- kops validate 명령으로 생성이 완료 되었는지 확인 할 수 있다.

~~~bash
$ kops validate cluster --name=${KOPS_CLUSTER_NAME}

~~~

- create cluster 했는데 생성이 안되면 기다려야한다. 클러스터 생성까지 5~10분 정도소요되기 때문이다.

## 클러스터 확인 및 수정

- 다음 명령어로 클러스터(master & node) 확인 할 수 있다.

~~~bash
$ kops get ig
~~~

- AWS 인스턴스들을 정지 시키고 싶을때 혹은 다시 실행 시키고 싶을 때

~~~bash
$ kops edit ig nodes
$ kops edit ig <master_name>
maxSize, minSize를 0으로 바꾸면 인스턴스 생성이 중지된다. 또는 1이나 2같은 숫자를 쓰면 그만큼 생성 된다.
~~~

- update cluster configuration

~~~bash
$ kops update cluster --yes
~~~

Rollling update

~~~bash
$ kops rolling-update cluster --yes
$ kops rolling-update cluster --cloudonly --force --yes
~~~

---

kops로 cluster를 구성하게되면 cluster의 정보가 amazon S3에 저장 되기 때문에 어느 환경에서든 kops와 kubectl만 깔려있으면 어디서든 cluster를 조작할 수 있다.
