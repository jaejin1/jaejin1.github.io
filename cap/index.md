# Availability vs consistency


일관성과 가용성에 대해서 알아보자.

<!--more-->

## availability vs consistency

### CAP theorem

분산 컴퓨터 시스템에서는 다음 중 2가지만 지원할 수 있다.

* Consistency 일관성
    
    * 모든 노드들은 같은 시간에 동일한 항목에 대하여 같은 내용의 데이터를 사용자에게 보여준다.
    * 모든 요청은 최신 데이터 또는 에러를 응답받는다.
    * Every read receives the most recent write or an error

* Availability 가용성

    * 모든 사용자들이 읽기 및 쓰기가 가능해야 하며, 몇몇 노드의 장애 시에도 다른 노드에 영향을 미치면 안된다.
    * 모든 요청은 정상 응답을 받는다.
    * Every request receives a response, without guarantee that it contains the most recent version of the information

* Partition Tolerance 분할내성

    * 노드간에 메시지 전달이 실패하거나 시스템 일부가 망가져도 시스템이 계속 동작할 수 있어야 한다.
    * The system continues to operate despite arbitrary partitioning due to network failures

하지만 네트워크는 신뢰할 수 없으므로 Partition Tolerance를 지원해야한다. 따라서 Consistency과 Availability 사이에 소프트웨어 절충안을 만들어야 한다.

#### CP - consistency and partition tolerance 

파티션된 노드의 응답을 기다리면 timeout 에러가 발생할 수 있다.

CP는 비즈니스 요구에 아주 작은 단위적으로 읽기 및 쓰기가 필요할 경우 좋은 선택이다.

#### AP - availability and partition tolerance

response로 모든 노드에서 가장 쉽게 사용 가능한 데이터를 반환하며 이는 최신 버전의 데이터가 아닐 수 있다. 만약 쓰기를 하고 모든 노드에 전파되는데 시간이 걸릴 수 있다.

AP는 비즈니스에서 궁극적인 Consistency를 허용? 하거나 외부 오류에도 불구하고 시스템이 계속 작동해야 하는 경우 좋은 선택이다.

## Consistency patterns

동일한 데이터의 multiple copies를 사용하면 클라이언트가 데이터를 일관되게 볼 수 있도록 동기화 하는 방법에 대한 옵션이 있다.

CAP theorem에서 consistency 정의는 최신 데이터 또는 에러를 응답받는다는 것을 기억하자.

### Weak consistency

`쓰기 후 읽으면 데이터를 볼 수도 있고 못볼 수도 있다.`

이것에 대한 최선의 접근 방식은 memcached(오픈 소스인 분산 메모리 캐싱 시스템)와 같은 시스템에서 볼 수 있다. 

~~memcached를 사용하면 memcached로 묶인 모든 서버는 동일한 가상 메모리 풀을 공유해서, 전체 클러스터에 대해서 동일한 위치에 저장되고 검색되어 진다.~~

또 VoIP, 화상채팅 및 실시간 멀티 플레이어 게임과 같은 실시간을 필요로 하는 서비스에서 잘 작동한다. 
예를들어 전화 통화시 몇 초 동안 수신이 끊긴 경우 다시 연결되면 연결이 끊겼을 때 말한 내용을 들을 수 없는 것처럼..

### Eventual consistency

`쓰기 작업의 대한 결과는 언젠가 확인 할 수 있다.`

쓰기 작업을 하게 되면 읽기는 결국 보게 된다. (일반적으로 milliseconds 이내에) 데이터는 비동기적으로 복제된다.

이런 접근은 DNS 및 Email과 같은 시스템에서 볼 수 있다. Eventual consistency는 고 가용성 시스템에서 잘 작동한다.

### Strong consistency

`쓰기 작업 이후 데이터를 본다.`

쓰기 작업 이후에는 읽기할 때 볼 수 있다. 데이터는 동기식으로 복제

이 접근은 RDBMS에서 볼 수 있다. Strong consistency는 Transaction이 필요한 시스템에서 잘 작동한다.

## Availability patterns

CAP theorem에서 Availability의 정의는 몇몇 노드의 장애 시에도 다른 노드에 영향을 미치면 안되고 모든 요청은 정상 응답을 받는다는 것을 다시 기억하고.

### Fail-over (장애 조치)

#### Active-passive

active passive 장애 조치를 사용하면, Active 서버와 Passive 서버간에 하트비트가 전송된다.

하트 비트가 중단되면 Passive 서버가 Active 서버의 IP를 인수하고 서비스를 다시 시작한다.

downtime의 길이는 Passive 서버가 이미 'hot' 대기에서 실행 중인지 'cold' 대기에서 시작하는지 여부에 따라 결정된다. 오직 Active 서버만 트래픽을 처리한다.

이는 master-slave 장애 조치라고도 한다.

#### Active-active

active-active 에서는 두 서버가 트래픽을 관리하여 부하를 분산 시킨다.

서버가 public facing인 경우 DNS는 public ip에 대해 알고 있어야한다. 

만약 internal facing인 경우 어플리케이션 login은 두 서버에 대해 알고 있어야한다.

active-active 장애 조치는 master-master 장애 조치라고도 한다.

#### Disadvantage(s): failover

* Fail over는 많은 하드웨어와 복잡성이 있다.
* 활성중인 시스템이 장애가 발생하면, 새로운 시스템이 띄워질 때 데이터를 복제하는 시간이 발생하므로 그때 데이터가 손실 될 가능성이 있다.

### Replication

* Master-slave replication
* master-master replication

### Availability in numbers

#### Availability in parallel vs in sequence

서비스가 실패하기 쉬운 여러 conponent로 되어 있는 경우, 서비스의 전체 availability는 component 가 sequence 한지 parallel한지에 따라 달라진다.

##### In sequence

availability < 100%인 두 component가 배치되면 전체 availability가 감소한다.

> Availability (Total) = Availability (Foo) * Availability (Bar)

Foo와 Bar의 Availability가 각각 99.9% 인경우 총 Availability는 99.8%가 된다.

##### In parallel

availability < 100% 인 두 component가 병렬로 연결 되면 전체 Availability는 증가한다.

> Availability (Total) = 1 - (1 - Availability (Foo)) * (1 - Availability (Bar))

Foo와 Bar의 Availability이 각각 99.%인 경우 총 Availability는 99.9999%가 된다.

---

**참고**

* https://github.com/donnemartin/system-design-primer#availability-in-numbers
