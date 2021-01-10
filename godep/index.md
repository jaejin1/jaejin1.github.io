# Go dependency management tool (dep)


Go 언어 에서는 [dep](https://github.com/golang/dep) 으로 의존성 관리를 하고 있었다. 

하지만

Once Go 1.11 is out, it really makes very little sense for a new project not already using dep to start using it. Dep has many, many problems - some of which Sam acknowledged in the talk - and you avoid all of them, avoid taking the time to master a system that is going away, and help make Go modules better by simply using Go modules from the start.

이렇다... 

<!--more-->

그래도 dep을 한번 사용해보고자 한다.

일단 go 의 폴더 구조를 다음과 같이 만들었다.

{GOPATH}
- bin
- pkg
- resources
- src
    - web

이후에 web이라는 프로젝트에서 진행했다.

먼저 dep을 설치한다.

~~~bash
$ go get -u github.com/golang/dep/cmd/dep

$ dep version

>>>
dep:
    version     : v0.5.0
    build date  : 2018-08-16
    git hash    : 224a564
    go version  : go1.10.3
    go compiler : gc
    platform    : darwin/amd64
    features    : ImportDuringSolve=false
~~~

설치가 되었으면 `dep init` 으로 시작해보자.

~~~bash
$ dep init
~~~

이 명령어를 실행하면 다음과 같은 파일들이 만들어진다.


- vender /

- Gopkg.toml

    Gopkg.toml은 초기화할 때 생성되는 파일로 dep의 동작을 감시하는 여러 규칙을 담고 있다.

- Gopkg.lock

    프로젝트의 의존성 그래프의 스냅숏을 가진 파일이다. 이 파일은 dep ensure 할 때 변경된다.

초기화 후 만들어진 Gopkg.toml 파일에 `gin` 을 사용하기 위해 다음 내용을 추가한다. 

[gin](https://github.com/gin-gonic/gin) 은 go 의 웹 프레임워크이다.

    [[constraint]]
      name = "github.com/gin-gonic/gin"
      branch = "master"

이제 추가한 의존성을 설치한다.

~~~bash
$ dep ensure
~~~

---

이건 수동으로 추가해주는 것이구 

~~~bash
$ dep ensure -add 설치할 모듈명
~~~

으로 추가가 가능한데

위에서 만든 web 프로젝트가 아닌 다른 프로젝트를 하나 만들고 다시

~~~bash
$ dep init
~~~

초기화를 한다. 

~~~bash
$ dep ensure -add github.com/gin-gonic/gin
~~~

이렇게 설치 하면 되는데 `no dirs contained any Go code` 에러가 나면 go 코드가 없어서 난 것이니 코드를 생성한 뒤 진행한다.

설치 하게 되면 자동으로 Gopkg.toml 과 Gopkg.lock에 gin 에 관한 것들이 설치 된다.
