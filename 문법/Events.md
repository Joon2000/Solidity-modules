### Events

Solidity에서 이벤트(Events)는 이더리움 블록체인에 로그를 기록하는 기능을 제공합니다.
이벤트는 스마트 계약과 블록체인 상호작용을 모니터링하고, 다양한 응용 프로그램에서 상태 변경을 추적하는 데 사용됩니다.
예를 들어, 사용자 인터페이스를 업데이트하거나, 블록체인 로그를 통해 특정 활동을 모니터링할 수 있습니다.

### 이벤트의 주요 특징

1. **블록체인 로그**: 이벤트는 트랜잭션 로그로 저장됩니다. 로그는 이더리움 블록체인에 저장되어 추적 가능하며, 이는 가볍고 저렴한 형태의 저장소 역할을 합니다.
2. **인덱싱**: 최대 3개의 매개변수를 인덱싱할 수 있습니다. 인덱싱된 매개변수는 로그를 필터링할 때 유용합니다.
3. **이벤트 청취**: 스마트 계약에서 발생한 이벤트는 웹 애플리케이션이나 다른 오프체인 서비스에서 청취(listen)할 수 있습니다.

### 이벤트 선언 및 사용법

#### 이벤트 선언

이벤트는 `event` 키워드를 사용하여 선언합니다. 선언 시 인덱싱할 매개변수를 `indexed` 키워드를 사용하여 지정할 수 있습니다.

```solidity
event 이벤트명(타입 indexed 매개변수명, 타입 매개변수명);
```

#### 예제 코드

다음 예제에서는 이벤트를 선언하고 사용하는 방법을 보여줍니다:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract Event {
    // 이벤트 선언
    // 최대 3개의 매개변수를 인덱싱할 수 있습니다.
    // 인덱싱된 매개변수는 로그를 필터링할 때 유용합니다.
    event Log(address indexed sender, string message);
    event AnotherLog();

    // 테스트 함수
    function test() public {
        // 이벤트 호출
        emit Log(msg.sender, "Hello World!");
        emit Log(msg.sender, "Hello EVM!");
        emit AnotherLog();
    }
}
```

### 예제 코드 설명

위 예제에서는 두 가지 이벤트를 선언하고 `test` 함수에서 호출하고 있습니다.

- **Log 이벤트**: 두 개의 매개변수를 가지며, 첫 번째 매개변수 `sender`는 인덱싱됩니다. 이 이벤트는 `msg.sender`와 메시지를 인자로 받아 로그를 기록합니다.
- **AnotherLog 이벤트**: 매개변수가 없는 이벤트입니다.

### 이벤트의 장점

1. **사용자 인터페이스 업데이트**: 스마트 계약에서 이벤트가 발생하면, 웹 애플리케이션에서 이를 감지하고 사용자 인터페이스를 동적으로 업데이트할 수 있습니다.
2. **저렴한 저장소**: 이벤트 로그는 저렴한 저장 공간을 제공합니다. 이벤트를 사용하면 상태 변수를 사용하지 않고도 블록체인에 정보를 저장할 수 있습니다.
3. **인덱싱 및 필터링**: 인덱싱된 매개변수를 사용하여 로그를 필터링할 수 있어, 특정 이벤트를 쉽게 추적할 수 있습니다.

### 예제 코드 설명

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract Event {
    // 이벤트 선언
    event Log(address indexed sender, string message);
    event AnotherLog();

    // 함수 내에서 이벤트 호출
    function test() public {
        emit Log(msg.sender, "Hello World!"); // 로그 기록
        emit Log(msg.sender, "Hello EVM!");   // 로그 기록
        emit AnotherLog();                    // 또 다른 로그 기록
    }
}
```

위 예제에서 `test` 함수가 호출되면, 두 개의 `Log` 이벤트와 하나의 `AnotherLog` 이벤트가 발생합니다. 각 이벤트는 블록체인에 로그로 기록되며, 나중에 이벤트 로그를 조회하거나 특정 이벤트 발생을
추적하는 데 사용할 수 있습니다.

### Remix 실습
 