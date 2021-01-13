# Domain name system


DNS (Domain Name System)은 www.example.com과 같은 도메인 이름을 IP 주소로 변환한다.

<!--more-->

DNS는 계층적이다. top level에 권한 있는 서버가 있다.

라우터 또는 ISP는 lookup을 할 때 연결할 DNS 서버에 대한 정보를 제공한다.

lower level DNS서버는 `전파 지연`으로 인해 문제가 생기는걸 방지해 매핑을 캐싱한다.

DNS결과는 TTL(Time to Live)에 의해 결정된 특정 기간 동안 브라우저 또는 OS에 의해 캐싱될 수 있다.

* NS 레코드 (name server)

    domain/subdomain에 대한 DNS 서버를 지정한다.

* MX 레코드 (maiil exchange)

    메시지를 수락하기위한 메일 서버를 지정한다.

* 레코드 (address)

    이름이 IP주소를 가리킨다.

* CNAME (표준[canocinal])

    이름을 다른 이름이나 CNAME(example.com에서 www.example.com으로) A 레코드를 가리킨다.

CloudFlare 및 AWS Route 53과 같은 서비스는 관리형 DNS 서비스를 제공한다.

{{< admonition note "참고" >}}
Route 53 및 DNS의 자세한 내용은 https://jaejin1.github.io/route53/ 참고
{{< /admonition >}}

일부 DNS서비스는 다양한 알고리즘으로 트래픽을 라우팅 할 수있다.

* Weighted round robin

    * Prevent traffic from going to servers under maintenance
    * Balance between varying cluster sizes
    * A/B testing

* Latency-based

* Geolocation-based

**DNS 단점**

* DNS서버에 액세스 하면 위에서 말한 것과 같이 캐싱을함에도 불구하고 약간의 지연이 발생 할 수 있다.

* DNS서버 관리는 복잡 할 수 있다. 일반적으로 정부, ISP 및 대기업에서 관리

* DNS서비스는 최근 DDoS공격을 받아 사용자가 Twitter의 IP주소를 알지 못하는 상태에서 Twitter와 같은 웹 사이트에 액세스하지 못하게 되었다.
