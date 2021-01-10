# Operating system in golang


Go 마스터하기 책을 공부하면서 기록한다.

<!--more-->

## 01. Go 언어와 운영체제

### 유닉스 stdin, stdout, stderr

유닉스 OS는 OS에서 실행되는 프로세스를 위해 항상 세 가지 파일을 열어 둔다.

유닉스는 모든 것을 파일 취급한다. 심지어 프린터나 마우스도..

유닉스에서는 양의 정수 값으로 된 `파일 디스크립터` 를 사용한다.

파일 디스크립터는 열린 파일에 접근하기 위한 내부 표현 수단으로, 긴 경로를 사용하는 것보다 훨씬 편리하다.

기본적으로 모든 유닉스 시스템은 /dev/stdin, /dev/stdout, /dev/stderr 세 가지의 특수한 표준 파일명을 사용한다. 각각에 대한 파일 디스크립터는 0, 1, 2이다. 이 세가지는 `표준 입력`, `표준 출력`, `표준 에러` 라고도 부른다. 또한 맥OS에서 파일 디스크립터 0은 /dev/fd/0을 통해 접근할 수 있고, 데비안 리눅스에서는 /dev/fd/0과 /dev/pts/0 둘 다 사용할 수 있다.

Go 코드에서 표준 입력은 os.Stdin, 표준 출력은 os.Stdout, 표준 에러는 os.Stderr로 접근할 수 있다. /dev/stdin, /dev/stdout, /dev/stderr 또는 이에 대한 파일 디스크립터 값을 직접 사용해도 되지만, Go 언어에서 제공하는 표준 파일명인 os.Stdin, os.Stdout, os.Stderr로 접근하는 것이 바람직하고 안전하며 이식성에 유리하다.

### 표준 출력 사용하기

~~~go
package main

import(
    "io"
    "os"
)
~~~

stdout을 이용 해보자. fmt 패키지 대신 io 패키지를 사용한다. os 패키지는 프로그램에서 커맨드 라인 인수를 읽고 os.Stdout에 접근하기 위해 사용한다.

~~~go
func main() {
    myString := ""
    arguments := os.Args
    if len(arguments) == 1 {
        myString = "Please give me one argument"
    } else {
        myString = arguments[1]
    }
~~~

myString 변수에 화면에 출력할 텍스트를 담는다. 

~~~go
    io.WriteString(os.Stdout, myString)
    io.WriteString(os.Stdout, "\n")
}
~~~

여기서 io.WriteString 함수는 fmt.Print() 함수와 똑같은 방식으로 작동하지만 두 개의 매개변수만 받는다는 점이 다르다. 첫 번째 매개변수는 쓰려는 파일을 지정한다. 여기서는 os.Stdout이고, 두 번째 매개변수는 string 타입 변수이다.

{{< admonition note "Note" >}}
is.WriteString 함수에서 첫 번째 매개변수의 타입은 반드시 io.Write여야 한다. 이 인터페이스를 사용하려면 두 번째 매개변수를 `바이트 슬라이스` 로 지정해야 한다. 하지만 string도 상관없다고 한다.
{{< /admonition >}}

~~~bash
$ go run test.go
Please give me one argument

$ go run test.go hi
hi

$ go run test.go hi hello
hi
~~~

### 표준 입력으로 부터 읽기

~~~go
func main() {
    var f *os.File
    f = os.Stdin
    defer f.Close()

    scanner := bufio.NewScanner(f)
    for scanner.Scan() {
        fmt.Printnl(">", scanner.Text())
    }
}
~~~

bufio.NewScanner()의 매개변수에 표준 입력을 지정해 호출하는 것을 볼 수 있다. 이 함수는 bufio.Scanner 변수를 리턴하는데, 이 값은 다시 Scan() 함수로 os.Stdin으로부터 한 줄씩 읽는 데 사용된다.

### 에러 출력

아까와 같은 코드를 보자. 한가지 다른점은 io.WriteString부분에 Stderr를 사용했다.

~~~go
package main

import(
    "io"
    "os"
) 

func main() {
    myString := ""
    arguments := os.Args
    if len(arguments) == 1 {
        myString = "Please give me one argument"
    } else {
        myString = arguments[1]
    }
    io.WriteString(os.Stdout, "This is Standard output \n")
    io.WriteString(os.Stderr, myString)
    io.WriteString(os.Stderr, "\n")
}
~~~

~~~bash
$ go run test.go hi

This is Standard output
hi
~~~

결과를 보면 표준 출력과 표준 에러를 구분할 수 없다. 

bash(1) 셸을 사용한다면 표준 출력 데이터와 표준 에러 데이터를 구분할 수 있다.

~~~bash
$ go run test.go 2>/tmp/stdError
This is Standard output

$ cat /tmp/stdError
Please give me one argument

$ go run test.go hi
This is Standard output
hi
~~~

에러 출력을 무시하려면 /dev/null 디바이스로 리디렉션하면 된다. 그러면 이 값을 무시한다.

~~~bash
$ go run test.go 2>/dev/null
This is Standard output
~~~

표준 출력과 표준 에러의 결과를 모두 같은 파일에 저장하고 싶다면, 표준 에러에 대한 파일 디스크립터(2)를 표준 출력에 대한 파일 디스크립터(1)로 리디렉션 하면된다. 

~~~bash
$ go run test.go >/tmp/output 2>&1
$ cat /tmp/output
This is Standard output
Please give me one argument
~~~

### 로그 파일 작성하기

log 패키지를 사용하면 시스템 로그로 로그 메세지를 보낼 수 있다. 이 패키지의 syslog라는 Go 패키지를 사용하면 log level과 logging facility 를 지정할 수 있다.

**로그 파일로 정보를 보내자**

~~~go
package main

import (
    "fmt"
    "log"
    "log/syslog"
    "os"
    "path/filepath"
)

func main() {
    programName := filepath.Base(os.Args[0])
    sysLog, err := syslog.New(syslog.LOG_INFO|syslog.LOG_LOCAL7, programName)
~~~

syslog.New() 함수의 첫 번째 매개변수로 우선순위를 지정한다.

여기에 로그 종류와 로그 수준을 한꺼번에 조합해서 표현한다. 예를 들어 LOG_NOTICE \| LOG_MAIL로 지정하면 로그 종류는 mail이고 로그 수준은 notice인 메시지를 보내게 된다.

또, 실제 수행하는 실행 파일의 이름을 지정하는 것이 바람직하다. 나중에 로그 파일을 보고 정보를 쉽게 찾기 위해서.

~~~go
    if err != nil {
        log.Fatal(err)
    } else {
        log.SetOutput(sysLog)
    }
    log.Println("LOG_INFO + LOG_LOCAL7: Logging in go")

    sysLog, err = syslog.New(syslog.LOG_MAIL, "Some program")
    if err != nil {
        log.Fatal(err)
    } else {
        log.SetOutput(sysLog)
    }
    log.Println("LOG_MAIL: Logging in go")
    fmt.Println("Will you see this?")
}
~~~

이 것을 실행하면 macOS에서는 

~~~bash
$ go runtest.go
Will you see this?
~~~

이렇게 나오고 로그 파일을 보면 로그가 나온다.

~~~bash
$ grep LOG_MAIL /var/log/mail.log

Jan  7 22:18:50 jaejin-ui-MacBook-Pro Some program[58311]: 2019/01/07 22:18:50 LOG_MAIL: Logging in go
~~~

**log.Fatal()**

log.Fatal()은 나쁜일이 발생해서 알려주자 마자 프로그램을 종료하고 싶을 때 사용한다.

**log.Panic()**

다시 실행될 수 없을 정도로 오류가 발생하는 순간, 관련된 정보를 최대한 알 수 있게 해주는 함수이다.
