# Service Discovery Pattern


MSA와 같은 분산 환경은 서비스간 원격 호출로 구성이 되고 IP와 Posrt를 이용하여 호출한다.

Cloud 환경에서의 인스턴스를 구축할 때 AutoScaling등으로 인한 IP, Port가 동적으로 변경될 일이 많다.

그래서 Client가 호출할 때 서비스 위치 (IP, Port)를 알아 낼 수 있는 기능이 필요한데, 이것을 Service Discovery라고 한다.

<!--more-->

---

**참고**

* https://bcho.tistory.com/1252
* https://mangchhe.github.io/springcloud/2021/04/07/ServiceDiscoveryConcept/
