## 목표

이더리움 블록체인에서 작동하는 간단한 Ether Wallet 스마트 계약을 작성
이 스마트 계약은 누구나 이더(ETH)를 보낼 수 있지만, 오직 소유자만이 출금할 수 있음
예제코드 실행 및 testnet에 배포

## 설명

Ether Wallet 스마트 계약은 Solidity 프로그래밍 언어로 작성된 간단한 계약입니다.
이 계약은 다음과 같은 기능을 합니다.

- 누구나 계약에 ETH를 보낼 수 있습니다.
- 오직 소유자만이 계약에서 ETH를 출금할 수 있습니다.
- 계약의 현재 잔액을 조회할 수 있습니다.

### 예제 스마트 계약 코드

다음은 Ether Wallet 스마트 계약의 전체 코드입니다.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract EtherWallet {
    address payable public owner;

    constructor() {
        owner = payable(msg.sender);
    }

    receive() external payable {}

    function withdraw(uint256 _amount) external {
        require(msg.sender == owner, "caller is not owner");
        payable(msg.sender).transfer(_amount);
    }

    function getBalance() external view returns (uint256) {
        return address(this).balance;
    }
}
```

### 코드 설명

#### 계약 선언 및 소유자 변수

```solidity
contract EtherWallet {
    address payable public owner;
```

`EtherWallet`이라는 계약을 선언하고, `owner`라는 `address payable` 타입의 변수를 선언합니다. 이 변수는 계약의 소유자 주소를 저장합니다.

#### constructor

```solidity
    constructor() {
        owner = payable(msg.sender);
    }
```

constructor는 계약이 배포될 때 한 번 호출됩니다. 여기서는 계약을 배포하는 주소(`msg.sender`)를 소유자로 설정합니다.

#### 이더를 받을 수 있는 함수

```solidity
    receive() external payable {}
```

`receive` 함수는 계약이 이더를 받을 수 있도록 합니다. 이 함수는 `external`과 `payable`로 선언되어, 외부에서 이더를 보낼 때 호출됩니다.

#### 출금 함수

```solidity
    function withdraw(uint256 _amount) external {
        require(msg.sender == owner, "caller is not owner");
        payable(msg.sender).transfer(_amount);
    }
```

`withdraw` 함수는 소유자만 이더를 출금할 수 있도록 합니다. `require` 문을 통해 호출자가 소유자인지 확인합니다. 출금할 금액은 `_amount` 매개변수로 전달됩니다.

#### 잔액 조회 함수

```solidity
    function getBalance() external view returns (uint256) {
        return address(this).balance;
    }
```

`getBalance` 함수는 계약의 현재 잔액을 반환합니다. 이 함수는 `view`로 선언되어 블록체인 상태를 변경하지 않습니다.

### Remix 실습

1. [Remix IDE](https://remix.ethereum.org/)에 접속합니다.
2. 새 파일을 만들어 위 예제 코드를 복사하여 붙여넣고 파일을 저장합니다.
3. 좌측 메뉴에서 "Solidity 컴파일러"를 이용해 코드를 컴파일하고, Deploy합니다.

##### 기능확인

1. 배포된 계약을 선택하고 "At Address"를 클릭하여 계약을 로드합니다.
2. "Receive" 함수를 사용하여 계약에 이더를 보냅니다.
3. "GetBalance" 함수를 호출하여 계약의 잔액을 확인합니다.
4. "Withdraw" 함수를 호출하여 소유자 주소로 이더를 출금합니다.
