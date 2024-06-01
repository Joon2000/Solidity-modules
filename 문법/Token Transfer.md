# Token Transfer

Solidity에서는 ERC-20과 ERC-721 같은 표준 인터페이스를 통해 토큰을 전송할 수 있다.

## ERC-20 토큰이란?

ERC-20은 이더리움 블록체인에서 토큰을 표준화하는 규격이다. 이 표준을 따르면, 서로 다른 토큰들이 동일한 방식으로 작동하게 되어 토큰 간의 상호 운용성이 높아진다. ERC-20 토큰은 주로 스마트 계약을 통해
발행되며, 기본적인 기능으로는 토큰 발행, 전송, 잔액 조회 등이 있다.

### 주요 기능

- `totalSupply`: 토큰의 총 공급량을 반환한다.
- `balanceOf`: 특정 주소의 토큰 잔액을 반환한다.
- `transfer`: 특정 주소로 토큰을 전송한다.
- `approve`: 토큰 전송을 승인한다.
- `allowance`: 승인된 토큰의 잔액을 확인한다.
- `transferFrom`: 승인된 토큰을 전송한다.

### ERC-20의 중요성

ERC-20 표준은 다음과 같은 이유로 중요하다:

1. 상호 운용성: 동일한 표준을 따르므로 서로 다른 ERC-20 토큰들이 동일한 방식으로 작동하여 다양한 플랫폼과 쉽게 통합될 수 있다.
2. 보안성: 표준화된 코드와 검증된 라이브러리를 사용하므로 보안성이 높다.
3. 유동성: 다양한 거래소에서 ERC-20 토큰을 지원하므로 유동성이 높다.

## 토큰 전송이란?

토큰 전송은 블록체인 네트워크 상에서 디지털 자산을 한 주소에서 다른 주소로 이동시키는 과정을 말한다. 토큰 전송의 일반적인 사용 사례는 다음과 같다:

### 1. 지불 및 결제

사용자는 토큰을 사용하여 상품이나 서비스 대금을 지불할 수 있다. 이는 전통적인 화폐를 사용하는 것과 유사하지만, 블록체인 기술을 통해 더 빠르고 저렴하게 거래할 수 있다.

### 2. 보상 시스템

플랫폼은 사용자에게 특정 활동에 대한 보상으로 토큰을 지급할 수 있다. 예를 들어, 콘텐츠 제작자는 플랫폼에서 콘텐츠를 게시함으로써 토큰을 받을 수 있다.

### 3. 투표 및 거버넌스

토큰 보유자는 스마트 계약을 통해 투표권을 행사하고, 프로젝트의 중요한 결정에 참여할 수 있다.

### 4. 자산의 디지털화

부동산, 예술품 등의 자산을 토큰화하여 블록체인 상에서 거래할 수 있다. 이는 자산의 유동성을 높이고, 소유권 이전을 더 쉽게 만든다.

### ERC-20 토큰 전송의 기본 원리

ERC-20 토큰 전송은 스마트 계약의 함수를 통해 이루어진다. 주요 함수는 다음과 같다:

- `transfer`: 이 함수는 보유자가 자신의 토큰을 다른 주소로 직접 전송할 때 사용된다. 예를 들어, A가 B에게 100개의 토큰을 전송하려면, A는 `transfer` 함수를 호출하고 B의 주소와
  100을 인자로 전달한다.

- `approve` 및 `transferFrom`: 이 두 함수는 제3자가 토큰을 전송하도록 허용하는 데 사용된다. 먼저, 토큰 소유자는 `approve` 함수를 사용하여 제3자에게 특정 양의 토큰을 전송할
  수 있는 권한을 부여한다. 그런 다음, 제3자는 `transferFrom` 함수를 사용하여 승인된 토큰을 전송할 수 있다.

## 미션 : Remix 실습

1. [Remix IDE](https://remix.ethereum.org/)에 접속한다.
2. 새 파일을 만들어 예제 코드를 입력한다.
3. 코드 작성 : 각 단계별로 코드를 작성한다.
4. 컴파일: 코드를 컴파일하여 오류 발생 여부를 확인한다.
5. 배포: 수정된 코드를 배포하고 배포된 계약 인스턴스를 사용하여 함수를 호출한다.
6. 결과 확인: 함수 호출 결과를 확인하고 예상된 동작과 실제 동작을 비교한다.

### 단계별 미션 수행

프로젝트 폴더 구조는 아래와 같다:

```
├── MyToken.sol
└── TokenTransfer.sol
```

미션은 두 개의 Solidity 파일을 작성하여 ERC-20 토큰을 생성하고 전송 기능을 구현하는 것이다.

#### 1. MyToken.sol 파일 작성

아래의 코드를 `MyToken.sol` 파일에 입력한다.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20 {
    constructor(uint256 initialSupply) ERC20("MyToken", "MTK") {
        _mint(msg.sender, initialSupply);
    }
}
```

- `contract MyToken is ERC20`: `MyToken` 계약은 `ERC20` 표준을 상속받는다.
- `constructor(uint256 initialSupply)`: 생성자 함수로, 초기 토큰 공급량을 설정한다.
- `_mint(msg.sender, initialSupply)`: 토큰을 발행하여 계약을 배포한 주소로 할당한다.

이 코드는 ERC-20 토큰의 기본적인 기능을 제공하는 스마트 계약을 정의한다. `ERC20` 표준을 상속받아 구현하며, 계약을 배포할 때 초기 공급량을 설정할 수 있다.

#### 2. TokenTransfer.sol 파일 작성

아래의 코드를 `TokenTransfer.sol` 파일에 입력한다.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "./MyToken.sol";

contract TokenTransfer {
    MyToken public token;

    constructor(address tokenAddress) {
        token = MyToken(tokenAddress);
    }

    function transfer(address recipient, uint256 amount) public returns (bool) {
        return token.transfer(recipient, amount);
    }

    function transferFrom(address sender, address recipient, uint256 amount) public returns (bool) {
        return token.transferFrom(sender, recipient, amount);
    }
}
```

- `import "./MyToken.sol";`: 로컬 파일 `MyToken.sol`을 임포트한다.
- `MyToken public token;`: `MyToken` 계약을 참조하는 변수이다.
- `constructor(address tokenAddress)`: 생성자 함수로, `MyToken` 계약의 주소를 받아서 초기화한다.
- `transfer(address recipient, uint256 amount)`: 토큰을 전송하는 함수이다.
- `transferFrom(address sender, address recipient, uint256 amount)`: 승인된 토큰을 다른 계정으로 전송하는 함수이다.

이 코드는 `MyToken` 계약의 주소를 받아서 토큰을 전송하는 기능을 제공한다. `transfer` 함수는 직접 토큰을 전송하고, `transferFrom` 함수는 승인된 토큰을 전송한다.

#### 3. 컴파일

`MyToken.sol` 파일과 `TokenTransfer.sol` 파일을 컴파일한다.

#### 4. 배포

1. `MyToken` 계약을 선택하고 `Deploy` 버튼을 클릭하여 배포한다. 초기 공급량을 입력하여 배포한다 (예: 1000).
2. 배포된 `MyToken` 계약 인스턴스의 주소를 복사한다.
3. `TokenTransfer` 계약을 선택하고, 생성자 매개변수로 `MyToken` 계약의 주소를 입력하여 배포한다.

#### 5. transfer 함수 테스트

1. `TokenTransfer` 계약 인스턴스에서 `transfer` 함수를 호출한다.
2. 수신자 주소와 전송할 토큰 양을 입력한다 (예: 0xRecipientAddress, 100).
3. `transfer` 함수 호출 후, 수신자 주소의 토큰 잔액을 확인하여 토큰이 전송되었는지 확인한다.

#### 6. transferFrom 함수 테스트

1. `MyToken` 계약에서 `approve` 함수를 호출하여 `TokenTransfer` 계약이 전송할 수 있는 토큰 양을 승인한다.
2. `TokenTransfer` 계약 인스턴스에서 `transferFrom` 함수를 호출한다.
3. 발신자 주소, 수신자 주소, 전송할 토큰 양을 입력한다 (예: 0xSenderAddress, 0xRecipientAddress, 50).
4. `transferFrom` 함수 호출 후, 수신자 주소의 토큰 잔액을 확인하여 토큰이 전송되었는지 확인한다.
