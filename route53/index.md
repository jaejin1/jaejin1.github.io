# Amazon Route53 DNS Service


Route53은 AWS에서 제공하는 DNS서비스 이다. 

DNS는 domain name(www.example.com) 을 ip주소로 바꿔 주는 일종의 dictionary 서비스 이다.

DNS동작과정은 네트워크 통신을 하기 위해 IP를 찾아가는 과정이다.

<!--more-->

이러한 mapping 정보를 저장해 놓는 파일을 DNS Zone file이라고 한다.

## 레코드

DNS서버에 저장해놓은 파일을 기반으로 주소를 변환하는데, 여기서 정의되는 레코드들은 다음과 같다.

1. A(Address Mapping records)

    도메인을 IP로 Mapping

    레코드 A는 주어진 호스트에 대한 IP주소 (IPv4)를 알려준다.  


2. AAAA(IP Version 6 Address records)

    레코드 AAAA(quad-A)는 주어진 호스트에 대해 IPv6주소를 알려준다.

    결국 A레코드와 같은 방식으로 작동. 차이는 IPv6이다.


3. CNAME(Canonical Name)

    도메인명을 다른 도메인과 Mapping할때 사용 (alias와 비슷)

    즉, 도메인 이름의 별칭을 만드는데 사용.  경우에 따라 CNAME 레코드를 제거하고 A레코드로 대체하면 성능 오버헤드를 줄일 수 있다고한다..(아직 잘 모르겠네요)


4. HINFO(Host Information)

    HINFO 레코드는 호스트에 대한 일반 정보를 얻는데 사용.

    CPU 및 OS유형을 알려준다. 따라서 두 호스트가 통신하기를 원할 때 OS특정 프로토콜을 사용할 수 있는 가능성을 제공한다. 일반적으로 보안 때문에 공용 서버에서 사용되지 않는다.


5. ISDN(Integrated Services Digital Network)

    ISDN주소를 알려준다. 

    ISDN주소는 국가코드, 국가 별 대상코드, ISDN 가입자 번호 및 선택적으로 ISDN 하위 주소로 구성된 전화 번호 이다.


6. MX(Mail exchanger)

    DNS도메인 이름에 대한 메일 교환 서버를 알려준다. 이 정보는 SMTP가 전자 메일을 적절한 호스트로 라우팅하는 데 사용한다. 일반적으로 DNS 도메인에 대해 둘 이상의 메일 교환 서버가 있으며 각 도메인에 우선 순위가 설정 된다.


7. NS(Name Server)

    DNS서버가 참조하는 다른 DNS서버이다. DNS 서버 자신에서 domain name에 대한 주소를 알아 내지 못할때, 이 NS레코드에 정의도니 서버로 가서 주소를 알아 온다.


8. PTR(Reverse-lookup Pointer records)

    정방향 DNS확인 (A, AAAA)레코드와 달리 PTR레코드는 IP주소를 기반으로 도메인 이름을 찾는데 사용된다.


9. SOA(Start of Authority)

    기본 이름 서버, 도메인 관리자의 전자 메일, 도메인 일련 번호 및 영역 새로 고침과 관련된 여러 타이머를 포함하여 DNS 영역에 대한 핵심 정보를 지정.


10. TXT(Text)

    형식이 지정되지 않은 임의의 텍스트 문자열을 저장할 수 있다. 

## 캐싱

DNS 서버의 특성중에 중요한 특성은 캐싱이다.

보통 DNS서버는 클라이언트가 사용하는 로컬 네트워크에 있는 DNS를 사용하게 된다. 이 DNS 서버들은 look up을 요청한 목적 서비스 서버에 대한 ip 주소를 다른 클라이언트가 요청할 때 응답을 빠르게 하기 위해 자체적으로 캐싱 하고 있다.

예를들어서 www.example.com 으로 접속할 수 있는 서비스가 있다고 하면 이 것을 운영하고 있는 회사의 DNS서버에 주소가 정의되어 있을 것이다. 만약 나의 스마트 폰으로 이 www.example.com 으로 접근하면, 내가 쓰는 통신사의 DNS서버를 이용해 주소를 look up할 것이고, 이 DNS서버는 위의 www.example.com 을 운영하고 있는 회사의 DNS서버에 주소를 물어보고 다음 서비스를 위해 캐시를 업데이트 한다.

이 캐시가 업데이트 되는 시간이 TTL이다.

## aws route53

route53에서 네임서버 등록하는 것을 알아보자.

1. route53사이트에서 Create Hosted Zone을 눌러서 생성하면 4개의 NS레코더가 생성된다.

    예를들어 다음과 같이 생성된다. 이를 도메인 등록 대행 기관 사이트에 가서 route53에서 지정된 NS정보를 입력한다.

    * ns-1.awsdns-50.com
    * ns-6.awsdns-20.net
    * ns-3.awsdns-30.co.uk
    * ns-8.awsdns-33.org


2. route53은 2가지 host zone이 있다.

    * Public Host zone

    일반 네임서버로 동작

    * Private Host zone

    AWS내에서만 동작


3. route53은 ALIAS가 있다

    도메인에 별칭을 줄 수 있어서 만약 example.com 도메인에 ALIAS Target을 www로 줘서 www.example.com 과 같은 IP를 응답주게 가능하다.


4. Route 기능 

    route53은 DNS + 모니터링 + L4 + GSLB기능을 제공한다.

    * Route53은 특정 포트에 대해 모니터링이 가능하다.
    * L4기능, Failover기능이 있다.
    * GSLB은 Global Server Load Balancing으로 지역에 상관없이 부하를 분산해주는 기능을 제공한다.

---

**참고**

* https://win100.tistory.com/360
* https://bcho.tistory.com/795
* https://brunch.co.kr/@topasvga/49
