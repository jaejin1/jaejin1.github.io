# Goroutines vs Threads


go routine과 thread를 비교해보자

<!--more-->

## goroutine thread의 차이점

### 메모리 소비

go routine은 생성하는데 많은 메모리가 필요하지 않다. 오직 2KB의 스택 공간만 필요하다. go routine을 할당하고 필요에 의해 힙 저장 공간을 확보하여 사용한다.
반면에 thread는 thread의 메모리와 다른 thread간의 보호 역할을 하는 Guard page라고 불리는 메모리 영역(1MB)과 함께 시작한다.

따라서 go routine은 작은 자원으로 생성할 수 있지만 Thread는 많이 생성하게 될 시 OutOfMemoryError가 발생할 수 있다. 

### 생성 및 삭제 비용

thread는 생성과 삭제 비용을 가진다. thread는 OS로 부터 리소스를 요청해야 하고 작업이 끝나면 리소스를 돌려 줘야하기 때문에..

### Context Switching 비용

thread가 blocking된다면 다른 thread가 작업을 가져와서 처리해야한다. 여기서 thread가 변경시 스케줄러는 모든 레지스터들을 save / restore해야한다. 16정도의 레지스터가 있음..

하지만 go routine은 스케줄링될 때 오직 3개의 레지스터만 save / restore 하면 된다. 

개수만 봐도 비용이 훨씬 적게 든다.

thread 보다 go routine을 많이 생성해서 그만큼 교체 비용이 발생할 수 있지 않냐라고 생각할 수 있겠지만 요즘 스케줄러들은 O(1)의 복잡도를 가진다고 한다. 따라서 교체시간이 go routine의 갯수에 영향을 받지 않다고 볼 수 있다.

## 런타임 스케줄러와 GMP

이제 차이점을 알았으니,, 어떻게 OS에서 제공하는 thread보다 더 적은 비용으로 go routine을 관리하는지 알아보자.

go routine은 GMP라고 부르는 구조에 의해 관리된다.


* G(go routine) : go routine의 구현체이며, 런타임에서 go routine을 관리하기 위해서 사용한다. blocking syscall등으로 go routine이 blocking될 경우 런타임은 다음 go routine을 실행하는데 이 과정에서 blocking이 최소화 된다.
* M(Machine) : OS의 thread를 의미한다.
* P(Processor) : process 자원을 의미하며, go를 실행할 때 `GOMAXPROCS`로 개수를 설정할 수 있다. default값은 시스템 코어 갯수. 정확히는 스케줄링에 대한 context를 지니고 있다.

### 스케줄러 작동 원리

먼저 프로그램이 실행되면 P가 할당되고 각각의 P에는 실행할 G가 배치된다. 보통 사용하는 `go func()` 같은 형태로 호출하면 새로운 go routine이 `P의 큐에 등록된다`. 

#### syscall이 발생했을 때(blocking)

만약 실행중 syscall이 발생하게 되면 해당 syscall을 인터럽트 하고 P로 부터 thread를 분리해두고 syscall이 재개 될 때 해당 작업을 다시 삽업하는 형태로 효율적으로 처린된다. (재개 되기 전까지 다른 thread로 넘겨(hadn off) 모든 go routine이 정상적으로 작동할 수 있도록 보장한다.) 

{{< admonition note "참고" >}}
이게 M에 P가 붙는 이유이다. 실행중인 M이 blocking되었을 때 다른 M으로 현재 상태를 그대로 넘겨야하기 때문에 P를 포함하고, 이는 `GOMAXPROCS`값이 1이더라도 go가 multi thread로 동작하는 이유이다.
{{< /admonition >}}

#### work-stealing scheduling

M은 P로부터 G를 가져와 실행하지만 P에 G가 없는 경우 무작위로 다른 P의 G의 절반을 훔치려고 하는데 이런 스케줄링 전략을 work-stealing scheduling이라고 불린다. 이를 통해 하나의 P에 대기열이 몰려있는걸 완화한다.

따라서 각 thread들은 놀지 않고 많은 작업을 더 효율적으로 동시에 처리 할 수 있게 된다.

## 결론

* syscall의 빈도를 줄여 유저 스페이스에서 동작하도록 하여 context switching을 최소화한다.
* thread 생성과 삭제의 빈도를 줄이고 기존 thread 최대한 재활용하여 OS 스레드 생성에 대한 비용을 줄인다.
* blocking syscall등으로 go routine이 blocking될 경우 Go 런타임은 다음 go routine을 실행하는데 이 과정에서 블로킹이 최소화되어 동시성 프로그래밍의 효율이 증가한다.

---

**참고**

* https://blog.nindalf.com/posts/how-goroutines-work/
* https://d2.naver.com/helloworld/0814313
* https://tech.ssut.me/goroutine-vs-threads/
