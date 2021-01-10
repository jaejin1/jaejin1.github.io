# Deploying smart contracts with truffle


스마트 컨트렉트를 쉽게 배포하기 위해서 truffle를 사용해보자.

<!--more-->

~~~bash
truffle init
~~~

초기화 완료되면

contracts, test, migrations 폴더가 생성되고 truffle.js, truffle-config.js가 생성된다.

contracts 폴더에는 solidity언어로 작성한 컨트랙트들을 두는곳 이다.

test폴더에는 컨트랙트를 테스트할 때 사용하는 파일을 두는 곳이다. 

truffle.js는 트러플에서 컨트랙트를 배포할 네트워크 설정, 컴파일러 언어 설정등 트러플과 관련된 설정을 하는 파일이다.

truffle-config.js 파일은 truffle.js를 설정할 예시를 보여주는 파일이다.

## 트러플 설정하기

truffle.js
~~~javascript
module.exports = {
    // See <http://truffleframework.com/docs/advanced/configuration>
    // to customize your Truffle configuration!
    networks: {
        development: {
            host: "172.17.0.6",
            port: "8545",
            network_id: "*",
        }
    }
};
~~~

geth 실행시에 설정한 값들을 집어 넣고

잘 동작하는지 확인하자
~~~bash
truffle console
~~~

또는
~~~bash
truffle console -network development
~~~

geth는 당연히 구동 상태여야한다.
~~~bash
truffle(development)> web3.eth.coinbase
'0xff53da3bf5403bbe86117a65db3c1e93f8adb9d5'
~~~

coinbase 주소가 나오면 잘 된거다.

## 컨트랙트 컴파일

트러플에서 컨트랙트를 컴파일 할 때 사용하는 명령어는 아래와 같다.
~~~bash
truffle compile
~~~

contracts/ 폴더 안에 들어있는 모든 파일이 컴파일 진행된다.

전에 작성했던

simple.sol을 컴파일해보자.

~~~solidity
pragma solidity ^0.4.23;

contract Simple {
        uint256 data;
        function set (uint256 _data)public {
                data = _data;

        }
        function get()public view returns(uint256){
                return data;
        }
}
~~~

~~~bash
root@d08fa5cce2b8:/home/truffle_example# truffle compile
Compiling ./contracts/Migrations.sol...
Compiling ./contracts/simple.sol...
Writing artifacts to ./build/contracts
~~~

build폴더를 만들고 해당 폴더에 컴파일이 완료된 정보가 담겨있는 파일들이 생성된다.

build/contracts 폴더가 생성되고 json파일이 생성된걸 볼 수 있다.

### 컨트랙트 배포하기

컨트랙트를 배포하기 위해서는 migrations폴더에 배포할 컨트랙트에 맞게 파일을 하나 작성해야한다.

migrations폴더 안에는 1_inital_migration.js 파일이 존재하는데

컨트랙트를 배포하기 위해서는 artifacts.require()로 배포를 원하는 컨트랙트의 정보를 획득 한 후 deployer.deploy()를 사용해서 배포를 하는 과정임을 알 수 있다. 수정안하는것이 좋다.

2_inital_migration.js 파일을 생성하고 
~~~javascript
var Simple = artifacts.require("Simple");

module.exports = function(deployer) {
    deployer.deploy(Simple);
};
~~~

작성해주고 배포를 해보자.!

geth는 miner.start()로 채굴시작하고
~~~bash
truffle migrate

Using network 'development'.

Running migration: 1_initial_migration.js
    Deploying Migrations...
    ... 0x896fe9568a631ba698daf89a6f8d2dbef1fe286f24c9e228ba58cc8d03134e33
    Migrations: 0x9d5d062e7cd54ce3b22f79e9433d41cdc8c253bb
Saving successful migration to network...
    ... 0x003250f43b24e185b6869feda9f1b1f1778f29aa1f3ac9add8115da18d449aa4
Saving artifacts...
Running migration: 2_initial_migration.js
    Deploying Simple...
    ... 0x4be336fbffff3ee8ec8ecd3eb8fee0c4bae0b913d032b2ac00dd95ad1861a81c
    Simple: 0x0d5c931ba28407a74e589940c1538792b017e6b2
Saving successful migration to network...
    ... 0x38f8ec8bd3de407410285d3aec267a989c038e049079ab793bc649458d129155
Saving artifacts...
~~~

이런 식의 결과가 나오면 성공이다.

gas limit 에러가 뜨면

truffle.js의 내용을
~~~javascript
module.exports = {
    // See <http://truffleframework.com/docs/advanced/configuration>
    // to customize your Truffle configuration!
    networks: {
        development: {
            host: "172.17.0.6",
            port: "8545",
            network_id: "*",
            gas: 2342414,
        }
    }
};
~~~

다음과 같이 gas를 추가해주고

어카운트 락 에러가 나면

언락을 해주면된다.

## 컨트랙트 사용하기
~~~bash
truffle console
~~~

들어가서
~~~bash
Simple.deployed().then((instance)=>{return instance.get.call()}).then((result) => { data = result });
~~~

를 입력해 data 변수에 컨트랙트에 저장된 값을 저장한다.
~~~bash
data.toNumber()
~~~

0의 값이 출력되면 성공 !!

Simple.deployed()를 호출하면 배포된 simple 컨트랙트의 객체가 반환된다. 해당 객체를 instance 변수로 받고 get함수를 호출해서 컨트랙트에 저장된 변수의 값을 가져온다.

컨트랙트이름.deployed()를 사용한다는점 기억하기.

set 함수를 사용해 변수에 데이터를 저장해보자.
~~~bash
Simple.deployed().then((instance) => { return instance.set(100) })
~~~

제대로 수행이 되면 geth에 정보가 출력된다.
~~~bash
> INFO [09-01|10:49:03.989] Submitted transaction                    fullhash=0xd901f598d41332a4a292bca74874c335672f26588bdd3567bd499d51f3626e5c recipient=0x0d5c931bA28407A74E589940c1538792b017e6B2
~~~

이제 mining을 하고 get함수를 실행 해보자.
~~~bash
Simple.deployed().then((instance)=>{return instance.get.call()}).then((result) => { data = result });
data.toNumber();
~~~

100이 출력 되면 성공이다  !

ex)
~~~javascript
Voting.deployed().then((instance)=>{return instance.getNumOfCandidates.call()}).then((result) => {data = result});
data.toNumber()

truffle(development)> Voting.deployed().then((instance)=>{return instance.addCandidate(0xc75711ce6897078ffd6af868fda11c8e9f0fc311,0xc75711ce6897078ffd6af868fda11c8e9f0fc312)})
{ tx: '0xb5adb6bd95815ffacd1197bc68685cbea87b92dc62fd6cd03997e756d8e43259',
    receipt:
    { blockHash: '0x0047122a32f44fb2a200fc9cc7f30e89dd7501989b288e10a7e6fffd22dd70b0',
        blockNumber: 339,
        contractAddress: null,
        cumulativeGasUsed: 90823,
        from: '0xff53da3bf5403bbe86117a65db3c1e93f8adb9d5',
        gasUsed: 90823,
        logs: [ [Object] ],
        logsBloom: '0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000008000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000200000000001000000000000000000000080000000008000000000000000000000000000000000000000000000000000000',
        root: '0x774c811955165298d0d2271c4285144f9915ec111749c1988c800df0c3a8aaa1',
        to: '0x5f9431c70262d23273f4d9ddb4f15d3906497dc9',
        transactionHash: '0xb5adb6bd95815ffacd1197bc68685cbea87b92dc62fd6cd03997e756d8e43259',
        transactionIndex: 0 },
    logs:
    [ { address: '0x5f9431c70262d23273f4d9ddb4f15d3906497dc9',
        blockNumber: 339,
        transactionHash: '0xb5adb6bd95815ffacd1197bc68685cbea87b92dc62fd6cd03997e756d8e43259',
        transactionIndex: 0,
        blockHash: '0x0047122a32f44fb2a200fc9cc7f30e89dd7501989b288e10a7e6fffd22dd70b0',
        logIndex: 0,
        removed: false,
        event: 'AddedCandidate',
        args: [Object] } ] }
~~~

---

unlock
~~~bash
personal.unlockAccount("0x6ea45f74c9803a9c3403c400975424e5825dfb70")
~~~
