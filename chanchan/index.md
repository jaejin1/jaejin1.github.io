# Golang chanchan


~~~go
channel := make(chan chan string)
~~~

go에 대해 channel은 대략 알겠지만 chan chan 이란 또 무엇인가..

<!--more-->

*chan chan*은 내가 준비되었다는 것을 알려줄테니, 내가 보낸 채널을 통해 보내줘 라고 한다.

이는 양반향 처리를 할 수 있다. 이해가 어려우니 코드를보자 !

~~~go
package main

import (
	"fmt"
	"time"
)

func main() {
	test := make(chan chan string)

	go test1(test)
	go test2(test)

	time.Sleep(time.Second)
}

func test1(channel chan chan string) {
	responsechan := make(chan string)

	channel <- responsechan

	response := <-responsechan

	fmt.Printf("Response: %v\n", response)
}

func test2(channel chan chan string) {
	responsechan := <-channel

	responsechan <- "ang !"
}
~~~

아ㅏ아ㅏ... 코드를 보는게 더 어렵다. 

보통 channel에는 type과 맞는 값을 전송하거나 받는데,

channel안에 channel을 전송하고 받는다고 생각하면 될 것 같다.

이를 통해 간단하게 user별로 여러개의 node를 생성한다고 가정하고 간단하게 작성해보자.

~~~go
package main

import (
	"fmt"
	"time"
)

func main() {
	User := make(chan chan string)
	User2 := make(chan chan string)

	go makenode(User)
	go input(User, "user1")

	go makenode(User2)
	go input(User2, "user2")
	go input(User2, "user2")

	time.Sleep(time.Second)
}

func makenode(user chan chan string) {
	makechannel := make(chan string)
	for {
		select {
		case user <- makechannel:
		case channelname := <-makechannel:
			fmt.Printf("channelname: %v\n", channelname)
		}
	}

}

func input(user chan chan string, username string) {
	responsechan := <-user

	responsechan <- "data " + username
}
~~~

User 별로 chan chan을 만들고, makenode go 루틴을 user 별로 실행.

data를 input하는 go 루틴 실행.

2번 실행했을때 결과는 다음과 같다.


~~~bash
channelname: data user2
channelname: data user2
channelname: data user1

---

channelname: data user1
channelname: data user2
channelname: data user2
~~~

chan chan을 보고 바로 작성한거라 특성에 맞지 않고 이상할 수 있겠지만 계속 해서 사용하면 괜찮지 않을까 싶다..

---

go의 chan chan을 비동기 처리에 사용해도 좋겠다는 생각을 가진다.

보통 mq를 사용하는데 간단하게 go는 chan chan을 사용해 비동기 처리에 이용할 수 있다.
