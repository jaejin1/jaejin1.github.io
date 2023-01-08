# Retry patterns


어플리케이션이 어떤 작업을 시도할 때 실패하게 되면 오류가 일시적인 부분이 많아 다시 요청을 하는 것이 좋다고 생각한다.

그에 따라 retry patterns라고 불리는 것들이 있었고 이를 정리해보려고 한다.

<!--more-->

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




AWS Go SDK를 사용하다가 보니 각 서비스별 [waiters](https://github.com/aws/aws-sdk-go/blob/main/service/codedeploy/waiters.go)가 있어 내부는 어떻게 동작하나 궁금해서 확인해본적이 있다.



---

**참고**

* https://blog.bytebytego.com/p/retry-patterns-episode-9
* https://aws.amazon.com/ko/blogs/architecture/exponential-backoff-and-jitter/
