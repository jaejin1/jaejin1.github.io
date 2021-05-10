# browser에서 url을 입력하면 무슨일이 일어날까?


웹 browser에 url을 입력하면 무슨일이 일어날지 정리해보자.

<!--more-->

## url을 주소창에 친다.

browser가 url을 해석하고, url문법에 따라서 요청을 하는데 만약 문법이 틀리다면 웹 browser의 기본 검색엔진으로 검색 요청을 한다.

url의 문법은 [URL 위키백과](https://ko.wikipedia.org/wiki/URL#cite_note-3) 에 자세히 나와있다.

## HSTS(HTTP Strict Transport Security) 목록을 로드해서 확인한다.

HSTS 목록에 있으면 첫 요청을 HTTPS로 보내고, 아닌경우 밑의 설명과 같이 HTTP로 보내서 HSTS 설정을 가져온다. (설정이 없다면 뭐 HTTP로 통신할 것이다.)

### HSTS란 무엇인지 알아보자.

일반적으로 HTTPS를 강제하게 될 때 서버측에서 302 redirection을 이용해 전환시켜주는데 이것이 취약점이 될 수 있다고 한다.

`http요청 -> 서버에서 302 redirect -> 클라이언트에서 https 재요청 -> 200 found`

따라서 클라이언트에게 강제로 HTTPS를 사용하도록 권장되는데 이것이 HSTS이다.

클라이언트에서 강제하기 때문에 http를 이용한 연결 자체가 최초부터 시도되지 않으며 클라이언트 측에서 차단된다는 장점이 있다. 

`https요청 -> 200 found`

사용자가 최초로 사이트에 접속시도를 하게 되면 웹서버는 HSTS설정에 대한 정보를 browser에게 응답하게 된다. browser는 이 응답을 근거로 일정 시간동안 HSTS응답을 받은 웹사이트에 대해서 https접속을 강제화 하게 된다.

실제 Chrome에서는 `chrome://net-internals/#hsts` 주소에서 확인 할 수 있다. 

![hsts](hsts.png "hsts")

또한 몇가지 웹사이트들은 browser자체에 등록되어 있는데 이를 확인하는 주소는 다음과 같다.

https://www.chromium.org/hsts

![preloaded-hsts](preloaded-hsts.png "preloaded-hsts")

### HSTS의 설정 방법

HSTS는 위에서 확인한 것과 같이 최초 요청은 서버에서 HSTS설정에 대한 정보를 가져 오기 때문에 서버에서 설정해 주어야 한다.

#### 예제

* apache httpd

> Header always set Strict-Transport-Security “max-age=86400; includeSubdomains; preload”

* nginx

> add_header Strict-Transport-Security “max-age=86400; includeSubdomains; preload”

설정을 보면 

* max-age=86400 : HSTS가 설정된 시간이며 초단위 이다.

* includeSubdomains : HSTS가 적용될 도메인의 subdomain까지 HSTS를 확장 적용함을 의미한다.

* preload : HSTS 적용이 클라이언트 측에서 preload로 이루어짐을 의미한다.

## DNS(Domain Name Server)를 조회한다.

1. DNS에 요청을 보내기 전에 먼저 browser에 해당 Domain이 cache돼 있는지 확인한다. DNS query가 이 곳에서 가장 먼저 실행된다.

또한 os 캐시를 확인하고, router 캐시를 확인하고 isp 캐시를 확인할 수도 있다.

2. 없을경우 local에 저장돼 있는 `/etc/hosts` 파일에서 참조할 수 있는 domain이 있는지 확인한다.

3. 1,2가 모두 실패 했을 경우 Network stack에 구성돼 있는 DNS로 요청을 보낸다. 

### DNS에 대해서 좀 더 자세히 알아보자.

DNS는 url들의 이름과 ip주소를 저장하고 있는 데이터 베이스이다. 모든 url들은 고유의 ip가 지정되어 있고 해당 ip를 통해서 서버에 접근할 수 있다.

ip를 확인하는 방법은 `nslookup www.google.com` 으로 확인할 수 있고 해당 ip를 browser에 쳐보면 google로 바로 접근 가능하다.

#### DNS로 요청을하면 해당 IP주소를 찾기위해서 동작하는 과정을 알아보자.

www.google.com과 같은 주소로 접속하고 싶으면 해당 ip를 반드시 알아야하는데, DNS query로 여러 다른 DNS 서버들을 검색해서 해당 사이트의 IP주소를 찾는 것이다. 
