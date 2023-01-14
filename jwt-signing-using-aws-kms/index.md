# JWT signing using AWS KMS


JWT 인증을 AWS KMS에서 key를 생성하여 이를 이용하여 token을 발급 받고 인증하는 것을 구현해보자

<!--more-->

기존엔 private key와 public key를 서버에다 생성하고 파일을 읽어서 token을 생성하고 인증을 했었다.

최근에 lambda로 api서버를 구성하면서 그런 구조는 이용할 수 없어 KMS를 이용해서 인증 처리를 진행해보고자 했었다.

## KMS Key 생성

![kms 생성](kms.png "kms 생성")

비대칭키, sign and verify를 선택하고 key는 RSA 2048을 선택하여 생성 한다.

![kms permissions](kms-permissions.png "kms permissions")

해당 key를 사용하는 AWS IAM user나 role을 설정 한다. 

보통은 user에서 발급받은 key를 사용하지 않고 assume role이나 instance profile을 사용할 것이기 때문에 Role만 설정 하는 것이 좋다고 생각한다.

입력값은 이정도로 넣고 생성하게되면, Public key를 web console에서 확인할 수 있고 파일로 다운로드 받을 수 있지만 이를 API 콜로 받아오는 걸 짜보려고 한다.

생성했으니 KMS Key ID를 기억해두자.


---


**참고**

* https://github.com/matelang/jwt-go-aws-kms

