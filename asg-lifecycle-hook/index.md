# ASG Lifecycle Hook


AWS Auto Scaling은 Lifecyle을 추가할 수 있는 기능을 제공한다.

Lifecycle 이벤트가 발생할 때 인스턴스에 대한 사용자 정의 작업을 수행할 수 있다.

<!--more-->

# Amazon EC2 Auto Scaling Lifecycle Hook

AWS EC2 ASG에서 인스턴스 시작 및 종료에 대한 이벤트를 잡아서 어떠한 작업을 수행하고 싶을 때가 있다.

예를 들어 

* ASG와 CodeDeploy가 설정되어 있고, ASG에서 Scale Out 이벤트가 발생 했을 때 CodeDeploy에서 배포가(인스턴스가 생성될 때 CodeDeploy 배포가 실행됨) 일어나기 전 어떤 스크립트를 실행하여 어플리케이션의 필요한 소프트웨어 패키지를 다운로드하여 트래픽 수신을 시작하기 전에 준비를 마치도록 할 수 있는 작업을 설정 할 수 있다.

* Sclae In 시점에 인스턴스가 종료되기 전 EventBridge로 인스턴스에 연결하여 종료되기 전에 로그파일을 다운로드 받는 로직을 추가할 수도 있고 Lambda를 이용해 추가 알림을 보낼 수 있는 작업을 할 수 있다.

AWS 문서에서 Lifecycle의 일반적인 용도는 인스턴스가 ELB에 등록되는 시점을 제어하는 것이라고 한다.

## Lifecycle Hook 작동 방식

![lifecycle hook](https://docs.aws.amazon.com/ko_kr/autoscaling/ec2/userguide/images/lifecycle_hooks.png "lifecycle hook")
출처 - https://github.com/donnemartin/system-design-primer#database

{{< admonition note "Note" >}}
작동 방식은 aws 공식 문서에서 상세하게 안내하고 있다.
https://docs.aws.amazon.com/ko_kr/autoscaling/ec2/userguide/lifecycle-hooks-overview.html
{{< /admonition >}}

## Lifecycle Hook 사용해보기

d

---

**참고**

* https://docs.aws.amazon.com/ko_kr/autoscaling/ec2/userguide/lifecycle-hooks.html
* https://docs.aws.amazon.com/ko_kr/autoscaling/ec2/userguide/lifecycle-hooks-overview.html
