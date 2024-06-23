# Payable

## 학습 목표
- Payable의 개념 및 특징을 이해한다.
- 실제 예제를 통해 Payable을 선언하고 사용하는 방법을 학습한다.

## Payable 개념 설명

Solidity에서 Payable은 smart contract에 ether를 받을 수 있게 하는 중요한 기능이다. Payable로 선언된 function과 address는 smart contract에 ether를 보낼 수 있다.

```solidity
pragma solidity ^0.8.24;

contract Fundraiser {
    // 외부에서 호출 가능한 function, ether를 전송할 수 있다.
    // 'payable' 키워드는 function이 ether를 받을 수 있음을 나타낸다.
    function donate() external payable {
        // Ether는 이 function이 호출될 때 함께 전송된다.
        // 이 smart contract의 잔고에 ether가 저장된다.
        // 여기서 받은 ether로 다른 address로 전송하는 등 추가 작업을 수행할 수 있다.
    }
}
```

## Payable의 특징

Payable은 다음과 같은 특징을 갖는다:

- **ether 수신 가능**: Payable function과 address는 ether를 받을 수 있다.
- **transfer 및 send 사용 가능**: Payable address는 transfer나 send를 통해 ether를 받을 수 있다.
- **ether 전송 처리**: ether 전송 실패 시 예외를 발생한다.

## 예제 코드

다음은 Payable을 사용한 간단한 예제다.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract Payable {
    // Payable address는 transfer나 send를 통해 ether를 보낼 수 있다.
    address payable public owner;

    // Payable constructor는 ether를 받을 수 있다.
    constructor() payable {
        owner = payable(msg.sender); // smart contract 소유자를 설정
    }

    // 이 smart contract에 ether를 입금하는 함수
    // 이 function을 ether와 함께 호출하면, smart contract의 잔고가 자동으로 업데이트된다.
    function deposit() public payable {}

    // 이 function을 ether와 함께 호출하면, 오류가 발생한다. 이 function은 Payable이 아니다.
    function notPayable() public {}

    // 이 smart contract에서 모든 ether를 인출하는 function
    function withdraw() public {
        // 이 smart contract에 저장된 ether의 양을 가져온다.
        uint256 amount = address(this).balance;

        // 모든 ether를 소유자에게 전송한다.
        (bool success,) = owner.call{value: amount}("");
        require(success, "Failed to send Ether"); // Ether 전송 실패 시 오류 발생
    }

    // 이 smart contract에서 입력된 address로 ether를 전송하는 function
    function transfer(address payable _to, uint256 _amount) public {
        // "_to"가 Payable로 선언되어 있는지 확인한다.
        (bool success,) = _to.call{value: _amount}("");
        require(success, "Failed to send Ether"); // Ether 전송 실패 시 오류 발생
    }
}
```

### 예제 코드 설명

#### constructor function

```solidity
constructor() payable {
    owner = payable(msg.sender);
}
```
- smart contract가 배포될 때 호출되며, ether를 받을 수 있는 constructor다.
- `msg.sender`를 `owner`로 설정한다.

#### 입금 function

```solidity
function deposit() public payable {}
```
- smart contract에 ether를 입금할 수 있는 function이다.
- `payable` 키워드로 인해 ether를 받을 수 있다.

#### 인출 function

```solidity
function withdraw() public {
    uint256 amount = address(this).balance;
    (bool success,) = owner.call{value: amount}("");
    require(success, "Failed to send Ether");
}
```
- smart contract에 있는 모든 ether를 소유자에게 전송한다.
- `address(this).balance`를 통해 현재 smart contract의 잔고를 가져온다.

#### 전송 function

```solidity
function transfer(address payable _to, uint256 _amount) public {
    (bool success,) = _to.call{value: _amount}("");
    require(success, "Failed to send Ether");
}
```
- 입력된 address로 ether를 전송한다.
- `_to`는 `payable`로 선언되어 있어야 한다.

## Remix에서 실습

1. Remix에서 새로운 Solidity 파일을 생성하고 예제 코드를 복사 붙여넣기 한다.
2. 예제 코드를 컴파일하고 배포한다.
3. 아래 함수들이 제대로 동작하는지 확인한다.

- deposit을 누르면 smart contract에 ether가 입금됐는지 확인할 수 있다. (10Ether로 초기값을 줘서 smart contract에 10ether 입금된 것을 확인할 수 있다.) <br>
  <img src="https://github.com/YoonHo-Chang/ludium-module/blob/789333214e6dc76a81b911b02dd06cb030ef1160/images/payabledeposit.png" width="250px" height="300px" title="interface" alt="interface">
- withdraw를 누르면 smart contract에서 모든 ether를 인출하는 것을 확인할 수 있다. <br>
  <img src="https://github.com/YoonHo-Chang/ludium-module/blob/789333214e6dc76a81b911b02dd06cb030ef1160/images/payablewithdraw.png" width="250px" height="300px" title="interface" alt="interface">
- owner를 누르면 smart contract의 소유자 address를 확인할 수 있다. <br>
  <img src="https://github.com/YoonHo-Chang/ludium-module/blob/789333214e6dc76a81b911b02dd06cb030ef1160/images/payableowner.png" width="250px" height="300px" title="interface" alt="interface">
- transfer를 누르면 주어진 주소로 ether를 전송할 수 있다.
- payable constructor를 주석처리하면 notPayable 상태라 오류가 발생한다.
