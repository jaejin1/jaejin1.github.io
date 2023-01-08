# Retry patterns


어플리케이션이 어떤 작업을 시도할 때 실패하게 되면 오류가 일시적인 부분이 많아 다시 요청을 하는 것이 좋다고 생각한다.

그에 따라 retry patterns라고 불리는 것들이 있었고 이를 정리해보려고 한다.

<!--more-->

![retry patterns](https://substackcdn.com/image/fetch/w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fbucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com%2Fpublic%2Fimages%2F8909aa28-5822-4347-862b-000bd334d8b6_2403x1140.jpeg "retry patterns")
출처 - https://blog.bytebytego.com/p/retry-patterns-episode-9

* Cancel
    
    요청 취소


AWS Go SDK를 사용하다가 보니 각 서비스별 [waiters](https://github.com/aws/aws-sdk-go/blob/main/service/codedeploy/waiters.go)가 있어 내부는 어떻게 동작하나 궁금해서 확인해본적이 있다.



---

**참고**

* https://blog.bytebytego.com/p/retry-patterns-episode-9
* https://aws.amazon.com/ko/blogs/architecture/exponential-backoff-and-jitter/
