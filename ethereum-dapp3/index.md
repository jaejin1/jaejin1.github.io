# Deploy smart contract


이전에 만들었던 ethereum network에 스마트 컨트랙트를 배포해보자.

<!--more-->

~~~bash
$ apt-get install solc
$ solc --version
~~~

simple.sol 같은 파일을 만들어서

~~~solidity
pragma solidity ^0.4.11;
contract Simple{
    uint256 data;
    function get() constant public returns(uint256) {
        return data;
    }
    function set (uint256 _data) public{
        data= _data;
    }
}
~~~

같이 저장하고 컴파일한다.
~~~bash

$ solc simple.sol

~~~

성공했으면 아무런 결과가 안나온다.

스마트컨트랙트 배포할때 필요한 것이 abi bin 이다.

~~~bash
root@9b88624ec1c4:/home# solc --abi simple.sol

======= simple.sol:Simple =======
Contract JSON ABI
[{"constant":false,"inputs":[{"name":"_data","type":"uint256"}],"name":"set","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"get","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}]
root@9b88624ec1c4:/home# solc --bin simple.sol

======= simple.sol:Simple =======
Binary:
608060405234801561001057600080fd5b5060df8061001f6000396000f3006080604052600436106049576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806360fe47b114604e5780636d4ce63c146078575b600080fd5b348015605957600080fd5b5060766004803603810190808035906020019092919050505060a0565b005b348015608357600080fd5b50608a60aa565b6040518082815260200191505060405180910390f35b8060008190555050565b600080549050905600a165627a7a72305820d55de6a6db862805a1312cf86887b07129afb5bd7a1313a1322c90bc1cb2a5070029
~~~

이걸 적어넣고 geth를 실행한다.

~~~bash
> var simpleAbi = [{"constant":false,"inputs":[{"name":"_data","type":"uint256"}],"name":"set","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"get","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}]
~~~

위의 값을 입력해준다.

~~~bash
> simpleAbi
[{
    constant: false,
    inputs: [{
        name: "_data",
        type: "uint256"
    }],
    name: "set",
    outputs: [],
    payable: false,
    stateMutability: "nonpayable",
    type: "function"
}, {
    constant: true,
    inputs: [],
    name: "get",
    outputs: [{
        name: "",
        type: "uint256"
    }],
    payable: false,
    stateMutability: "view",
    type: "function"
}]
~~~

잘나온다.

binarycode도 설정해줘야한다. 

유의할점은 16진수임을 알려주기위해 앞에 `0x` 를 추가한다.

~~~bash
> var simpleBin = "0x608060405234801561001057600080fd5b5060df8061001f6000396000f3006080604052600436106049576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806360fe47b114604e5780636d4ce63c146078575b600080fd5b348015605957600080fd5b5060766004803603810190808035906020019092919050505060a0565b005b348015608357600080fd5b50608a60aa565b6040518082815260200191505060405180910390f35b8060008190555050565b600080549050905600a165627a7a72305820d55de6a6db862805a1312cf86887b07129afb5bd7a1313a1322c90bc1cb2a5070029"
undefined
> simpleBin
"0x608060405234801561001057600080fd5b5060df8061001f6000396000f3006080604052600436106049576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806360fe47b114604e5780636d4ce63c146078575b600080fd5b348015605957600080fd5b5060766004803603810190808035906020019092919050505060a0565b005b348015608357600080fd5b50608a60aa565b6040518082815260200191505060405180910390f35b8060008190555050565b600080549050905600a165627a7a72305820d55de6a6db862805a1312cf86887b07129afb5bd7a1313a1322c90bc1cb2a5070029"
~~~

잘되었다.

이제 컨트랙트를 만들어보자.

eth.contract()를 이용해 abi를 설정해줘야한다. 

그 후에 컨트랙트를 배포하는 계좌가 누구인지, 얼마만큼 가스를 소비할 것인지, 배포할 컨트랙트의 데이터가 무엇인지 명시해줘야한다.

~~~bash
> var simpleContract = eth.contract(simpleAbi)
undefined
> var simpleTransferObject = {from: eth.accounts[0], data: simpleBin, gas: 2000000};
undefined
~~~

배포하기전에 첫번째 계좌에서 가스를 소비해야하니 락을 풀어주자

~~~bash
personal.unlockAccount(eth.accounts[0]);
~~~

simpleContract.new(simpleTransferObject)를 입력하면 배포가 시작된다. 

~~~bash
> var contractObj = simpleContract.new(simpleTransferObject);
INFO [08-25|05:22:07.056] Submitted contract creation              fullhash=0x788543be4447cee4f518fdc077d20c88b69a7993d4d0358794a1b46a810134d6 contract=0xAB2274abf1E4712BBA53FfBA1C53bA1e54fbeb4d
undefined
~~~

contractObj 객체를 통해 컨트랙트 정보를 얻을 수 있다.

~~~bash
> contractObj
{
    abi: [{
        constant: false,
        inputs: [{...}],
        name: "set",
        outputs: [],
        payable: false,
        stateMutability: "nonpayable",
        type: "function"
    }, {
        constant: true,
        inputs: [],
        name: "get",
        outputs: [{...}],
        payable: false,
        stateMutability: "view",
        type: "function"
    }],
    address: undefined,
    transactionHash: "0x788543be4447cee4f518fdc077d20c88b69a7993d4d0358794a1b46a810134d6"
}
~~~

address란이 undefined인데 이건 채굴이 아직 안되서 결정이 안되어있는 것이다.

채굴한뒤에 확인해보면 

~~~bash
> contractObj
{
    abi: [{
        constant: false,
        inputs: [{...}],
        name: "set",
        outputs: [],
        payable: false,
        stateMutability: "nonpayable",
        type: "function"
    }, {
        constant: true,
        inputs: [],
        name: "get",
        outputs: [{...}],
        payable: false,
        stateMutability: "view",
        type: "function"
    }],
    address: "0xab2274abf1e4712bba53ffba1c53ba1e54fbeb4d",
    transactionHash: "0x788543be4447cee4f518fdc077d20c88b69a7993d4d0358794a1b46a810134d6",
    allEvents: function(),
    get: function(),
    set: function()
}
~~~

address가 할당 된 것을 볼 수 있다.

컨트랙트를 사용해보자

~~~bash
> contractObj.set(1000, {from:eth.accounts[0], gas:1000000})
INFO [08-25|05:25:40.183] Submitted transaction                    fullhash=0xbe21e8d2107d79a9dcfa0f922398b2d239b48441be8a57de3b8ae1d55750db02 recipient=0xAB2274abf1E4712BBA53FfBA1C53bA1e54fbeb4d
"0xbe21e8d2107d79a9dcfa0f922398b2d239b48441be8a57de3b8ae1d55750db02"
~~~

set함수를 이용한다. 그후 채굴을 하면

~~~bash
> contractObj.get()
1000
~~~

잘 동작한다.

### 배포된 컨트랙트 사용하기

배포된 컨트랙트를 사용하기 위해서는 컨트랙트의 abi와 주소값이 필요하다. eth.contract 함수에 abi값을 인자로 넘겨준다.

~~~bash
> contract2 = eth.contract([{"constant":false,"inputs":[{"name":"_data","type":"uint256"}],"name":"set","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"get","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"}]
... )
{
    abi: [{
        constant: false,
        inputs: [{...}],
        name: "set",
        outputs: [],
        payable: false,
        stateMutability: "nonpayable",
        type: "function"
    }, {
        constant: true,
        inputs: [],
        name: "get",
        outputs: [{...}],
        payable: false,
        stateMutability: "view",
        type: "function"
    }],
    eth: {
    accounts: ["0x241530b417837cdcce8036a7b7f095995509d5c9", "0xca09b7785f4179988e14ee87cfe7e93933a002c7"],
    blockNumber: 24,
    coinbase: "0x241530b417837cdcce8036a7b7f095995509d5c9",
    compile: {
        lll: function(),
        serpent: function(),
        solidity: function()
    },
    defaultAccount: undefined,
    defaultBlock: "latest",
    gasPrice: 18000000000,
    hashrate: 0,
    mining: false,
    pendingTransactions: [],
    protocolVersion: "0x3f",
    syncing: false,
    call: function(),
    contract: function(abi),
    estimateGas: function(),
    filter: function(options, callback, filterCreationErrorCallback),
    getAccounts: function(callback),
    getBalance: function(),
    getBlock: function(),
    getBlockNumber: function(callback),
    getBlockTransactionCount: function(),
    getBlockUncleCount: function(),
    getCode: function(),
    getCoinbase: function(callback),
    getCompilers: function(),
    getGasPrice: function(callback),
    getHashrate: function(callback),
    getMining: function(callback),
    getPendingTransactions: function(callback),
    getProtocolVersion: function(callback),
    getRawTransaction: function(),
    getRawTransactionFromBlock: function(),
    getStorageAt: function(),
    getSyncing: function(callback),
    getTransaction: function(),
    getTransactionCount: function(),
    getTransactionFromBlock: function(),
    getTransactionReceipt: function(),
    getUncle: function(),
    getWork: function(),
    iban: function(iban),
    icapNamereg: function(),
    isSyncing: function(callback),
    namereg: function(),
    resend: function(),
    sendIBANTransaction: function(),
    sendRawTransaction: function(),
    sendTransaction: function(),
    sign: function(),
    signTransaction: function(),
    submitTransaction: function(),
    submitWork: function()
    },
    at: function(address, callback),
    getData: function(),
    new: function()
}       
~~~

at함수에 컨트랙트의 주소를 설정하면된다.

~~~bash
> var contractObj2 = contract2.at("0xab2274abf1e4712bba53ffba1c53ba1e54fbeb4d")
undefined
> contractObj2.get()
1000
~~~

위에 address를 가져와서 실행한결과로 1000이 잘 출력된다.
