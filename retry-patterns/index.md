# Retry patterns


어플리케이션이 어떤 작업을 시도할 때 실패하게 되면 오류가 일시적인 부분이 많아 다시 요청을 하는 것이 좋다고 생각한다.

그에 따라 retry patterns라고 불리는 것들이 있었고 이를 정리해보려고 한다.

<!--more-->

MSA 구조에서는 서로 다른 서버와 통신을 할 일이 많다.

서로 다른 서버와 통신할 때 최대한 실패를 줄이는게 중요하고 서비스 비지니스 로직을 모두 실패 처리하거나 fallback 처리하는 것은 큰 리소스 낭비가 될 수 있어서 일시적인 오류나 네트워크 이슈의 경우 retry를 하여 실패를 줄일 수 있다.


## Retry patterns

![retry patterns](https://substackcdn.com/image/fetch/w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F8909aa28-5822-4347-862b-000bd334d8b6_2403x1140.jpeg "retry patterns")
출처 - https://blog.bytebytego.com/p/retry-patterns-episode-9

서버의 어떤 문제 때문에 요청이 실패 하고 재요청을 하게 되었을 때 많은 Client들이 있다면 시스템에 과부하가 걸릴 수 있다. 

이로 인해 재시도의 시간을 제어하거나등의 작업으로 서비스 과부하를 방지 한다.

* Cancel
    
    요청 취소

* Immediate retry

    Client쪽에서 즉시 재시도

* Fixed intervals

    실패 후 일정 시간 후에 재시도

* Incremental intervals

    실패 후 일정 시간 후에 재시도 하고 또 실패발생시 점점 대기시간을 늘림

* Exponential backoff

    실패 후 재시도 사이 대기 시간을 2배씩 늘림 1초, 2초, 4초... 이런식으로

* Exponential backoff with jitter

    재시도 할 때 어떠한 randomness을 줘서 재시도 함

일반적으로는 점진적으로 대기 시간이 늘어나는 Exponential Backoff 전략을 많이 사용한다고 하는데, 이 또한 여러 트래픽이 동시에 몰린다면 똑같은 시간 간격으로 모두 재시도를 하기 때문에 결국 과부하가 걸릴 수도 있을 것이다.

따라서 `Jitter`라는 개념이 등장했다.

Jitter는 원래 통신 언어로 패킷 지연이 일정하지 않고 수시로 변하면서 간격이 일정하지 않다는 용어라고 한다.

간단하게는 randomness을 줘서 요청하는 시간대를 분산시키는 것이라고 보면 될 것 같다.

Jitter 자체도 여러 알고리즘이 있고, 이를 aws쪽에서 정리 해놓은 문서가 있다.

* Equal Jitter
* Decorrelated Jitter
* Full Jitter
* ...

https://aws.amazon.com/ko/blogs/architecture/exponential-backoff-and-jitter/


## aws go sdk 내부 waiters

AWS Go SDK를 사용하다가 보니 각 서비스별 [waiters](https://github.com/aws/aws-sdk-go/blob/main/service/codedeploy/waiters.go)가 있어 내부는 어떻게 동작하나 궁금해서 확인해본적이 있다.

waiter들은 내부적으로 15초 간격으로 최대 120번 요청을 해서 완료나 실패가 될 때까지 기다리는데, 개발하다보니 30분 이상으로 대기해야할 때가 있어서 커스텀해서 사용하려고 내부를 보니 위에서 말한 retry와는 다르지만 지속적으로 요청을 하기 떄문에 어떤 시간 간격으로 보내는지 확인해봤다.

```go
func (w Waiter) WaitWithContext(ctx aws.Context) error {

	for attempt := 1; ; attempt++ {
		req, err := w.NewRequest(w.RequestOptions)
		if err != nil {
			waiterLogf(w.Logger, "unable to create request %v", err)
			return err
		}
		req.Handlers.Build.PushBack(MakeAddToUserAgentFreeFormHandler("Waiter"))
		err = req.Send()

		// See if any of the acceptors match the request's response, or error
		for _, a := range w.Acceptors {
			if matched, matchErr := a.match(w.Name, w.Logger, req, err); matched {
				return matchErr
			}
		}

		// The Waiter should only check the resource state MaxAttempts times
		// This is here instead of in the for loop above to prevent delaying
		// unnecessary when the waiter will not retry.
		if attempt == w.MaxAttempts {
			break
		}

		// Delay to wait before inspecting the resource again
		delay := w.Delay(attempt)
		if sleepFn := req.Config.SleepDelay; sleepFn != nil {
			// Support SleepDelay for backwards compatibility and testing
			sleepFn(delay)
		} else {
			sleepCtxFn := w.SleepWithContext
			if sleepCtxFn == nil {
				sleepCtxFn = aws.SleepWithContext
			}

			if err := sleepCtxFn(ctx, delay); err != nil {
				return awserr.New(CanceledErrorCode, "waiter context canceled", err)
			}
		}
	}

	return awserr.New(WaiterResourceNotReadyErrorCode, "exceeded wait attempts", nil)
}
```

코드를 보면 attempt에 따라 Delay를 걸고 재 요청하는 것으로 확인했다.

굳이 위의 retry pattern으로 따지자면 Incremental interval 쪽으로 속하려나..

물론 위에서 다뤘던 retry하는 것과는 다르다.

https://github.com/aws/aws-sdk-go/blob/main/aws/request/waiter.go#L195

---

**참고**

* https://blog.bytebytego.com/p/retry-patterns-episode-9
* https://aws.amazon.com/ko/blogs/architecture/exponential-backoff-and-jitter/
