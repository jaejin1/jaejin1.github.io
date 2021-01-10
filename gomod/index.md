# Go modules (mod)


go 1.11 버전에서는 go modules 가 도입되어 시험적으로 사용할 수 있게 되었다.

<!--more-->

## GO111MODULE

go 1.11 버전에서 go modules가 등장하며 기존 `GOPATH` 와 `vendor/` 에 따라 동작하던 go 커맨드와의 공존을 위한 `GO111MODULE` 이라는 임시 환경변수가 생성되었다.

- on

    go 커맨드는 `GOPATH` 에 관계없이 go modules의 방식 대로 동작한다.

- off

    go modules은 전혀 사용되지 않고 기존에 사용되던 방식대로 `GOPATH` 와 `vendor/` 를 통해 go 커맨드가 동작한다.

- auto

    `GOPATH/src` 내부에서의 `go` 커맨드는 기존의 방식대로 go 커맨드는 go modules의 방식대로 동작한다.

여기서는 on으로 사용해본다.

~~~bash
export GO111MODULE=on
~~~

소스코드를 생성하고 소스코드 안에 모듈경로를 추가하면 간단히 `go mod init` 커맨드를 실행하면 된다. 

go.mod 파일이 만들어지면서 폴더구조는 다음과 같다.

{project}
- main.go
- go.mod

`go.mod` 파일에는 현재 모듈에 대한 정보와 코드에서 사용하고 있는 외부 패키지에 대한 의존성 정보가 담기게 된다.

이제 build 해보자.

~~~bash
$ go build
~~~

자동으로 import된 패키지를 찾고 `GOPATH/pkg/mod/` 디렉토리 하위에 버전에 따라 생성된다.

프로젝트 디렉토리에 `go.sum` 파일도 생성되는데, 이 파일은 설치된 모듈의 해시 값을 저장해두고, 매 go 커맨드가 실행되기 전에 설치 되어있는 모듈의 해시값과 go.sum에 저장된 해시 값을 비교하여 설치된 모듈의 유효성을 검증한다.

다시 한번 폴더구조는

{project}
- main.go
- go.mod
- go.sum
- *.exe

새로운 모듈을 추가하고 싶다면 `go get <module-path>@<module-query>` 커맨드를 이용하면 된다. 

버전 지정 필요가 없다면 그냥 코드에 바로 import 시키고 `go` 명령어를 사용하면 된다.
