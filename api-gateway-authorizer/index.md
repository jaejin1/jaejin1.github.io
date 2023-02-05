# API Gateway Authorizer


AWS Lambda로 끄적이다가 API Gateway를 붙여보자! 해서 보던 중 API Gateway는 인증 및 권한부여 기능을 제공 해주고 있었고 그 중 Authorizers의 존재를 봐서 api를 위한 Lambda 호출 전에 인증을 위한 Lambda를 호출 하는 것을 구성해봤다.

<!--more-->

---

**참고**

* https://docs.aws.amazon.com/ko_kr/autoscaling/ec2/userguide/lifecycle-hooks.html
* https://docs.aws.amazon.com/ko_kr/autoscaling/ec2/userguide/lifecycle-hooks-overview.html
