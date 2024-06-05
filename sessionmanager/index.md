# AWS Session Manager


공격의 표면, Bastion을 관리 하는 운영상의 부담 및 발생하는 추가 비용을 더욱 줄이기 위해, AWS Systems Manager Session Manager를 사용하여, Bastion을 운영하거나 조작 할 필요없이 EC2 인스턴스에 안전하게 연결 할 수 있다.

Bastion실행에 수반되는 불편함과 인스턴스의 inbound SSH port 개방으로 유발되는 위험을 피하고자 한다.

<!--more-->

* 보안 액세스

    * 수동으로 인스턴스에 사용자 계정, 암호 또는 SSH를 설정할 필요가 없으며 inbound port를 개발하지 않아도 된다.

    * Session Manager는 SSM 에이전트를 사용하여 인스턴스에서 시작한 암호화된 채널을 통해 인스턴스와 통신하며 Bastion이 필요하지 않다.

* 액세스 제어

    * IAM 정책과 사용자를 사용하여 인스턴스에 대한 액세스를 제어하며 SSH key를 배포하지 않아도 된다. 

    * IAM의 날짜 조건 연산자를 사용하면 원하는 시간/유지 관리 기간으로 액세스를 제한 할 수 있다.

* 감사 용이성

    * 명령과 응답은 Amazon CloudWatch 및 S3 버킷에 로깅 할 수 있다.

    * 또한 세션이 시작될 때 SNS 알림을 수신할 수 있도록 설정할 수 있다.


## Session Manager의 작동방식

### Session Manager 사용을 위한 Role

Session Manager를 사용하여 EC2 인스턴스에 액세스 하려면 인스턴스에 SSM 에이전트가 실행 중이어야 한다.

또한 인스턴스에는 AmazonEC2RoleForSSM 정책을 부여해 주어야 한다.


### SSM Role을 갖는 EC2 인스턴스를 Public Subnet, Private Subnet에 설치

기본적으로 Session Manager의 기능을 사용하기 위해서는 EC2 인스턴스가 인터넷을 통해 접속이 가능하여야 한다.

(SSM 에이전트는 Session Manager의 public Endpoint에 연결할 수 있어야 하므로)

따라서 Public Sebnet에 설치하거나, Private Subnet에 설치할 경우 PrivateLink 연결을 설정해서 사용할 수 있다.


#### Public Subnet에 설치할 경우

Instance에 공인 IP를 부여 받고, SSM 에이전트를 설치 하고, IAM정책만 부여해주면 System Manager에 Session Manager메뉴에 해당 EC2 인스턴스를 확인 할 수 있을 것이다.


#### Private Subnet에 설치할 경우

Public과 동일하지만 인스턴스가 인터넷에 접속 가능해야 하기 때문에 PrivateLink 설정을 하고 Public Subnet 구성과 같이 하면 된다.

**Info Notice:** 
```
VPC End pont를 생성할시 Service Name에서 다음 4가지를 추가 해줘야 한다.

1. "com.amazonws.region-name.ec2"

2. "com.amazonaws.region-name.ec2messages"

3. "com.amazonaws.region-name.ssm"

4. "com.amazonaws.region-name.ssmmessages"
```

## AWS Session Manager과 Bastion 비교 

|   | Session Manager | Bastion |
|---:|---:|---:|
| 비용 | Session Manager는 별도의 요금 없음 <br> [AWS PrivateLink 요금](https://aws.amazon.com/ko/privatelink/pricing/)  | Instance 운영 비용 |
| 로그 | Amazon CloudWatch 및 S3 버킷 제공 <br> SNS 알림 | log 파일 수집 -> log 수집기 보내는 작업 필요 |
| 보안 | 수동으로 인스턴스에 사용자 계정, 암호 또는 SSH 키를 설정할 필요 없음. <br> Session Manager Console에 접속하면 인스턴스 접속 가능 (계정 관리 따로 안해도 됨 IAM으로 user에 따른 권한 부여) | Bastion 서버에서 instance접속 SSH key관리 필요 |
| 설정 | 인스턴스에 SSM에이전트 설치 <br> 인스턴스가 생성될때 AmazonEC2RoleforSSM IAM 정책 할당 <br> Session Manager에 대한 권한 IAM정책 수립 필요 ([참고](https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/getting-started-restrict-access-examples.html)) <br> VPC에 PrivateLink 설정 필요 | 구성된 Bastion 서버에 맞게 설정 |

---

**참고**

* https://aws.amazon.com/ko/privatelink/pricing/
* https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/session-manager.html
