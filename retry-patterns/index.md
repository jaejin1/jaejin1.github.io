# Retry patterns


어플리케이션이 어떤 작업을 시도할 때 실패하게 되면 오류가 일시적인 부분이 많아 다시 요청을 하는 것이 좋다고 생각한다.

그에 따라 retry patterns라고 불리는 것들이 있었고 이를 정리해보려고 한다.

<!--more-->

AWS Go SDK를 사용하다가 보니 각 서비스별 [waiters](https://github.com/aws/aws-sdk-go/blob/main/service/codedeploy/waiters.go)가 있어 내부는 어떻게 동작하나 궁금해서 확인해본적이 있다.



---

**참고**

* https://blog.bytebytego.com/p/retry-patterns-episode-9
* https://aws.amazon.com/ko/blogs/architecture/exponential-backoff-and-jitter/
