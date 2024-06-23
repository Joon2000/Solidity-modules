Sending Ether
=============

목표
----
- Ether를 보내고 받을 때 사용하는 함수에 대해서 배운다.
- 기본적인 함수 구조를 익힌다.
- 각 함수의 용도를 이해하고 실습해 본다.

개념
-----
### Ether를 전송할 때

1. transfer
   - transfer()함수는 2300gas가 소모되고, 실패시 error를 발생시킨다.
   - 간단하고 직관적인 방식으로, 실패시 자동으로 롤백되는 안전한 방법이다.

```solidity
address payable recipient = payable(0xRecipientAddress);
recipient.transfer(amount);
```  

2. send
   - send()함수는 2300gas가 소모되고, bool형으로 성공 여부를 return된다.
   - 실패시 에러를 처리할 코드가 따로 필요하다.
  
```solidity
address payable recipient = payable(0xRecipientAddress);
bool success = recipient.send(amount);
require(success, "전송 실패");
```
  
3. call
   - call()함수는 모든 가스 혹은 설정된 가스를 소모하고, bool형으로 성공 여부를 return한다.
   - 이더를 전송하는 가장 유연한 방법으로, 임의의 데이터를 포함할 수 있고 가스 한도를 지정할 수 있다.
   - re-entrancy-attack을 방지하기 위한 조치가 필요하다.
     - 다른 contract를 호출하기 전에 모든 상태를 변경한다.
     - re-entrancy 방지를 위한 modifier를 사용한다.

re-entrancy-attack 이란, 공격자가 재귀적으로 인출을 수행하여 Contract에 있는 모든 Ether를 훔치는 공격이다.

<br/>

- call의 기본 구조는 다음과 같다.
```solidity
(bool success, byte memory data) = someAddress.call{value: msg.value, gas: 5000}(abi.encodeWithSignature("someFunction(uint256)",123));
```
1. `someAddress`: 호출할 주소
2. `{value: msg.value, gas: 5000}`: 옵션 블록(전송할 Ehter 양, 호출에 사용되는 가스의 양)
3. `abi.encodeWithSignature("someFunction(uint256)",123)`: 인코딩된 함수 서명과 매개변수


<br/>

다음은 modifier를 이용해서 re-entrancy 공격을 방지하여 call함수를 호출하는 과정이다.

```solidity
bool private locked;

modifier noReentrant() {
    require(!locked, "re-entrancy 없음");
    locked = true;
    _;
    locked = false;
}

function sendEther(address payable recipient, uint amount) external noReentrant {
    (bool success, ) = recipient.call{value: amount}("");
    require(success, "전송 실패");
}
```

<br/>

### Ether를 받을 때

1. recieve() external payable
   - recieve()의 경우는 msg.data, 즉 calldata가 비어있는 경우 사용된다.
2. fallback() external payable
   - 이외의 경우(calldata가 존재할 경우)에는 fallback()함수가 사용된다.

```solidity
event Log(string func, uint gas);

receive() external payable{
    // msg.data가 비어있는 경우
    emit Log("receive", gasleft());
}

fallback() external payable{
    // msg.data가 존재하는 경우
    emit Log("fallback", gasleft());
}

```

예외)
만약 Contract에 fallback()함수만 있는 경우에는 msg.data가 비어 있어도 fallback() 함수가 자동으로 실행된다.
  

<br/>
  
예제코드
-------

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract ReceiveEther {
    /*
    어떤 함수가 호출 되는가?

           Ether 전송
               |
         msg.data가 비어있는가?
              / \
            예    아니오
            /       \
    receive()의 유무  fallback()
         /   \
       있다    없다
       /        \
    receive()   fallback()
    */

    // msg.data가 비어 있을 때 Ether를 받는 함수.
    receive() external payable {}

    // msg.data가 있을 때 Ether를 받는 함수.
    fallback() external payable {}

    // balance 확인
    function getBalance() public view returns (uint256) {
        return address(this).balance;
    }
}

contract SendEther {
    function sendViaTransfer(address payable _to) public payable {
        // 현재는 Ether를 전송하는데 이 함수를 사용하는 것을 권장하지 않는다.
        _to.transfer(msg.value);
    }

    function sendViaSend(address payable _to) public payable {
        // send()함수는 성공 여부를 bool 값으로 return한다.
        // transfer 함수와 마찬가지로 Ether를 전송하는데 이 함수를 사용하는 것을 권장하지 않는다.
        bool sent = _to.send(msg.value);
        require(sent, "Failed to send Ether");
    }

    function sendViaCall(address payable _to) public payable {
        // call()함수는 성공 여부를 bool 값으로 return한다.
        // 현재는 Ether를 전송하는데 이 함수를 사용할 것을 권장한다.
        (bool sent, bytes memory data) = _to.call{value: msg.value}("");
        require(sent, "Failed to send Ether");
    }
}

```

#### sendViaCall 함수

```solidity
function sendViaCall(address payable _to) public payable {
    (bool sent, bytes memory data) = _to.call{value: msg.value}("");
    require(sent, "Failed to send Ether");
}
```
- `_to` 주소로 `msg.value`만큼의 이더리움을 전송한다.
- `bool sent`: 이더리움 전송의 성공 여부를 나타낸다. 성공하면 `true`, 실패하면 `false`를 반환한다.
- `bytes memory data`: 함수의 반환 데이터를 포함하는 `bytes` 배열이다.
- `require(sent, "Failed to send Ether")`: 호출이 실패할 경우 트랜잭션을 롤백하고 오류 메시지를 출력한다.



Remix에서 실습
--------

1. Remix에서 새로운 solidity 파일을 생성해서 예제 코드를 복사 붙여넣기 한다.
2. 컴파일 후 ReceiveEther contract와 SendEther를 각각 deploy한다.
3. 아래 버튼들이 제대로 동작하는지 확인한다.

- deploy한 ReceiveEther contract의 주소를 복사하여 Ether를 전송시킬 함수에 붙여넣는다.

#### 함수 호출

- VALUE를 1 Ether로 설정한 후 함수를 호출한다.

<img src="/images/sending-ether/ether.png" width="40%" height="70%">


- 호출하면 ReceiveEther contract의 Balance가 1ETH가 된 것을 확인할 수 있고 getBalance를 호출해서 Balance를 확인할 수도 있다.

<img src="/images/sending-ether/transfer.png" width="40%" height="70%">

- 같은 방식으로 send 함수와 call 함수를 실행시켜 확인해본다.

<img src="/images/sending-ether/send.png" width="40%" height="70%">
<img src="/images/sending-ether/call.png" width="40%" height="70%">









