# Load balancer


로드 밸런서는 클라이언트 요청을 애플리케이션 서버 및 데이터베이스와 같은 곳에 리소스를 분배한다. 

<!--more-->

로드 밸런서는 다음 3개에 대해서 효과적이다.

* Preventing requests from going to unhealthy servers
* Preventing overloading resources
* Helping to eliminate a single point of failure

로드밸런서는 하드웨어 또는 HAProxy와 같은 소프트웨어로 구성할 수 있다.

또 다른 기능으로는 

* SSL termination
     
    들어오는 요청을 해독하고, 서버 응답을 암호화 하여 백엔드 서버가 cost가 많이 드는 작업을 수행 할 필요가 없다.

    각 서버에 X.509 인증서를 설치할 필요가 없다.

* Session persistence

    Issue cookies and route a specific client's requests to same instance if the web apps do not keep track of sessions

장애로 부터 보호하기 위해 active-passive or active-active 의 여러 로드밸런서를 설정하는게 일반적이다.

로드 밸런서는 다음과 같은 다양한 지표를 기반으로 트래픽을 라우팅 할 수 있다.

* Random
* Least loaded
* Session/cookies
* round robin or weighted round robin
* Layer 4
* Layer 7


### Layer 4 load balancing

Layer 4 로드 밸런서는 transport layer에서 어떻게 요청을 분산할지 정한다.

일반적으로 여기서는 헤더의 소스, 대상 IP 주소 및 port가 포함되지만 패킷 내용은 포함 되지 않는다.

Layer 4 로드 밸런서는 NAT를 수행하여 업스트림 서버와 네트워크 패킷을 전달한다.

### Layer 7 load balancing

Layer 7 로드 밸런서는 application layer에서 어떻게 요청을 분산할지 정한다.

여기에서는 헤더, 메시지 및 쿠키의 내용이 포함 될 수 있다.

또한 네트워크 트래픽을 terminate 하고 메시지를 읽고 로드 밸런싱 결정을 내린 다음 선택한 서버에 연걸한다.

예를 들어 비디오를 호스팅하는 서버로 트래픽을 보낸다고 하면 민감한 사용자 청구 트래픽 같은 것을 보안이 강화된 서버로 보낼 수 있다.


### Horizontal scaling

로드 밸런서는 Horizontal scaling에 도움이 되어 성능과 가용성을 향상 시킬수 있다. 반대로 Vertical Scaling(수직 확장)은 단일 서버의 머신을 좋은 성능으로 올리는 것. 에 비해 비용이 효율적이고 가용성이 높다. 

#### 단점

수평적으로 확장하면 복잡성이 발생하고 서버 복제를 해야함.

* 서버의 상태를 저장하지 않아야함. 사용자 관련 데이터를 포함하면 안된다.
* 세션은 DB(SQL,NoSQL) 또는 Redis와 같은 중앙 집중식 데이터 저장소에 저장될 수 있다.

캐시 및 데이터베이스와 같은 다운 스트림 서버는 업스트림 서버가 확장됨에 따라 더 많은 동시 연결을 처리해아한다.

### 로드 밸런서 단점

* 리소스가 충분하지 않거나 제대로 구성이 안되어 있으면 성능 병목 현상이 일어난다.
* Introducing a load balancer to help eliminate a single point of failure results in increased complexity.
* 단일 로드밸런서는 단일 장애지점이라서 여러 로드 밸런서를 구성하면 복잡성이 더욱 증가한다.

---

**참고**

* https://github.com/donnemartin/system-design-primer#load-balancer
