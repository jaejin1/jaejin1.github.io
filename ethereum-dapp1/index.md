# Setup private ethereum blockchain network


docker를 이용해서 이더리움 노드 3개와 웹서버 1개를 만들고 웹서버에서 회원가입을 하고 투표를 하는 정보들을 이더리움 네트워크에 저장시켜보는 실습을 해본다.

<!--more-->

~~~bash
$ docker pull ubuntu  
$ docker run -it -p 10001:10001 ubuntu
~~~

ubuntu os에 설치하려고한다. 이유는 설치하는 과정을 보여주려고....

~~~bash
$ apt-get install software-properties-common
$ apt-get install software-properties-common python-software-properties
$ add-apt-repository -y ppa:ethereum/ethereum
$ apt-get update
$ apt-get install ethereum
~~~

이후에 

~~~bash
$ geth
$ geth version

>>>
WARN [08-17|08:21:11.660] Sanitizing cache to Go's GC limits       provided=1024 updated=666
Geth
Version: 1.8.13-stable
Git Commit: 225171a4bfcc16bd12a1906b1e0d43d0b18c353b
Architecture: amd64
Protocol Versions: [63 62]
Network Id: 1
Go Version: go1.10.1
Operating System: linux
GOPATH=
GOROOT=/usr/lib/go-1.10
~~~

라고 하면 잘 실행 될 것이다.

아래의 명령어를 사용하면 geth console에 명령어를 입력 할 수 있다.

~~~bash
$ geth --dev console
~~~


---

준비할 것이 2가지 있다.

1. private network에서 사용할 계좌
2. 제네시스 블록 정보.

## 계좌 생성하기

~~~bash
$ geth --datadir "./data" account new
~~~

—datadir 옵션은 network가 구동될 때 블록정보, 계좌정보를 저장할 폴더를 지정하는 옵션이다.

~~~bash
$ ls data/keystore/

>>>
UTC--2018-08-17T08-24-17.766131786Z--49a13ee73d2fc8c1578ce1512530aa1724ee42b7
~~~

계좌주소 출력하기

~~~bash
$ geth --datadir "./data" account list

>>>
WARN [08-17|08:26:02.392] Sanitizing cache to Go's GC limits       provided=1024 updated=666
INFO [08-17|08:26:02.392] Maximum peer count                       ETH=25 LES=0 total=25
Account #0: {49a13ee73d2fc8c1578ce1512530aa1724ee42b7} keystore:///home/data/keystore/UTC--2018-08-17T08-24-17.766131786Z--49a13ee73d2fc8c1578ce1512530aa1724ee42b7
~~~

다음과 같이 쓰면 생성된 계좌의 리스트가 출력된다.

## 제네시스 블록 생성하기

private network를 구동하기 위해서는 첫 번째 블록을 생성해야한다.

~~~json
{
    "config": {
        "chainId": 15,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
    "difficulty": "200000000",
    "gasLimit": "2100000",
    “coinbase”: "0x{위에 생성한 계좌주소}",
    "alloc": {
        "0x{위에 생성한 계좌주소}": { "balance": "1000000000000000000000000" },
        }
}
~~~

제네시스 블록을 만들때 필요한 json 파일의 구성이다.

1. config는 생성할 private network의 버전을 정하는 설정이다. chainId, homesteadBlock, eip155Block, eip158Block 모두 이더리움 속성과 관련된 설정이다.

    chainId는 replay protection을 사용하는 설정이다. homestead는 이더리움의 두 번째 메인 릴리즈를 사용하기 위해 설정하는 속성이고 0으로 설정하면 해당 릴리즈를 사용한다. eip155은 gas 비용에 관련된 하드포커 설정이며 eip158은 state cleanig과 관련된 설정이다.

2. difficulty 는 이더리움에서 채굴의 난이도를 설정하는 옵션
3. gas limit 한 블록이 담을 수 있는 gas의 수치를 말한다. gas 수치가 높을 수록 한 블록이 담을 수 있는 거래가 많아진다. 
4. coinbase는 network를 구동할 때 기본으로 사용할 계좌를 설정하는 옵션 ( 채굴할 때 코인을 받을 계좌 )
5. alloc 는 private network를 구동할 떄 미리 계좌에 이더를입금 시킬지를 설정하는 옵션이다. 설정하기 원하는 계좌와 이더 값을 명시한다.

제네시스 json파일을 생성후

~~~bash
$ geth --datadir "./data" init "./genesis.json"

>>>
WARN [08-17|08:31:55.253] Sanitizing cache to Go's GC limits       provided=1024 updated=666
INFO [08-17|08:31:55.254] Maximum peer count                       ETH=25 LES=0 total=25
INFO [08-17|08:31:55.254] Allocated cache and file handles         database=/home/data/geth/chaindata cache=16 handles=16
INFO [08-17|08:31:55.261] Writing custom genesis block
INFO [08-17|08:31:55.261] Persisted trie from memory database      nodes=1 size=150.00B time=41.229µs gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [08-17|08:31:55.261] Successfully wrote genesis state         database=chaindata                 hash=e47869…3ca16c
INFO [08-17|08:31:55.261] Allocated cache and file handles         database=/home/data/geth/lightchaindata cache=16 handles=16
INFO [08-17|08:31:55.264] Writing custom genesis block
INFO [08-17|08:31:55.264] Persisted trie from memory database      nodes=1 size=150.00B time=37.247µs gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [08-17|08:31:55.265] Successfully wrote genesis state         database=lightchaindata                 hash=e47869…3ca16c
~~~

성공적으로 genesis block이 생성 되었다고 출력된다.

## Private network 구동하기

첫 번째 블록을 만들었으니 네트워크 실행해보자.

private network를 구동할 때 유의해야 할 옵션은 —networkid이다. networkid를 1로 지정시 메인 이더리움에 연결이 되고 2는 테스트 네트워크로 사용중이다. 따라서 1과 2를 제외한 수를 지정해야 한다.

또한 --identity 옵션을 사용해 네트워크에 이름을 지정할 수 있다. networkid와 identity는 node들 간에 연결 할 때 사용되는 옵션이다.

그리고 --mine 옵션을 설정해서 geth를 구동했을 때 채굴을 할 수 있게 한다. --nodiscover 옵션을 사용해서 다른 노드를 찾는 설정을 끈다. 

 
~~~bash
$ geth --networkid 1988 --identity "mynetwork" --datadir "./data" --nodiscover console
~~~

아래와 같이 콘솔창이 나오면 성공이다.

~~~bash
>>>
... 생략
INFO [08-17|08:35:18.788] Etherbase automatically configured       address=0x49A13ee73D2FC8C1578cE1512530Aa1724EE42b7
coinbase: 0x49a13ee73d2fc8c1578ce1512530aa1724ee42b7
at block: 0 (Thu, 01 Jan 1970 00:00:00 UTC)
    datadir: /home/data
    modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0

>
~~~

eth.accounts를 입력하면 현재 생성되어 있는 계좌들이 출력된다.

~~~bash
> eth.accounts
["0x49a13ee73d2fc8c1578ce1512530aa1724ee42b7"]
~~~

eth.accounts[0]은 첫번째 생성된 계좌를 출력

~~~bash
> eth.accounts[0]
"0x49a13ee73d2fc8c1578ce1512530aa1724ee42b7"
~~~

이더 잔고를 조회 할때는 eth.getBalance()함수를 사용하고 인자로 계좌 주소를 적어준다.

~~~bash
> eth.getBalance(eth.accounts[0])
1e+24
~~~

새로운 계좌를 만들어보면

~~~bash
> personal.newAccount()
Passphrase:
Repeat passphrase:
"0xaefa6e0f4b91fd67e31498f82f34ae95762aac71"
> eth.accounts
["0x49a13ee73d2fc8c1578ce1512530aa1724ee42b7", "0xaefa6e0f4b91fd67e31498f82f34ae95762aac71"]
~~~

두개가 출력된걸 확인 할 수 있다.

이제 이더를 옮겨보자 

반드시 해야하는 작업이 이더를 전송하려는 계좌의 락을 풀어줘야한다.!!!

~~~bash
> personal.unlockAccount(eth.accounts[0])
Unlock account 0x0a39454d85c9ff43f5aba2025da36d67ccfa50d9
Passphrase:
true
~~~

이제 보내보자

~~~bash
> eth.sendTransaction({from: eth.accounts[0], to:eth.accounts[1], value:100})
INFO [08-18|06:27:14.550] Submitted transaction                    fullhash=0xd592ae1540ae288979caa5fe8bcfc4bc41c625190b978bb505587b241fdb346e recipient=0xa6ae554222c1Ac7cB98eEF8abFd3b8f8CD9C2817
"0xd592ae1540ae288979caa5fe8bcfc4bc41c625190b978bb505587b241fdb346e"
~~~

전송을 해도 

잔고에 변화가 없다.

~~~bash
> eth.getBalance(eth.accounts[0])
1e+24
> eth.getBalance(eth.accounts[1])
0
~~~

블록 체인이기 때문에 채굴이 되어야 발생한 거래가 적용이 된다. 채굴을 해보자

~~~bash
$ miner.start()
$ minet.stop()

INFO [08-18|06:32:51.972] Generating DAG in progress               epoch=1 percentage=97 elapsed=3m4.253s
INFO [08-18|06:32:53.809] Generating DAG in progress               epoch=1 percentage=98 elapsed=3m6.089s
INFO [08-18|06:32:56.898] Generating DAG in progress               epoch=1 percentage=99 elapsed=3m9.179s
INFO [08-18|06:32:56.905] Generated ethash verification cache      epoch=1 elapsed=3m9.184s
INFO [08-18|06:35:20.852] Successfully sealed new block            number=1 hash=a9c709…37cf8b
INFO [08-18|06:35:20.866] 🔨 mined potential block                  number=1 hash=a9c709…37cf8b
INFO [08-18|06:35:20.869] Commit new mining work                   number=2 txs=0 uncles=0 elapsed=3.373ms
INFO [08-18|06:35:36.470] Successfully sealed new block            number=2 hash=7f96ee…bf362a
INFO [08-18|06:35:36.473] 🔨 mined potential block                  number=2 hash=7f96ee…bf362a
~~~

Generating DAG in progress 가 반복되다가 블럭이 생성되는 경우가 있다.!!

~~~bash
> eth.getBalance(eth.accounts[0])
1.0000099999999999999999e+24
> eth.getBalance(eth.accounts[1])
100
~~~

이후에 이더가 전송된 것을 확인 가능하다.!!

---

### 간단한 geth 명령어는 다음과 같다.

블록 갯수 체크

~~~bash
> eth.blockNumber
~~~

채굴에 성공시 보상받는 계좌 조회

~~~bash
> eth.coinbase
~~~

다른 계정으로 옮기고 싶을때

~~~bash
> miner.setEtherbase(eth.accounts[1])
> eth.coinbase
~~~

진행 기다리는 트랜잭션 확인

~~~bash
> eth.pendingTransactions
[{
    blockHash: null,
    blockNumber: null,
    from: "0x241530b417837cdcce8036a7b7f095995509d5c9",
    gas: 90000,
    gasPrice: 18000000000,
    hash: "0x8aff56402bbbe8547dca097ba443a3d668bbac4cd73477e8613fc2c74ca8b002",
    input: "0x",
    nonce: 1,
    r: "0xc029343dac203271bee3de5d73a586e8740a547342b61f3aebdbbbb0ac4bd0a8",
    s: "0x27112e6e766ddfccf55123d057fe130f7b39e99f76b36c0aefa140168cdd36f9",
    to: "0xca09b7785f4179988e14ee87cfe7e93933a002c7",
    transactionIndex: 0,
    v: "0x41",
    value: 1000
}]

~~~
