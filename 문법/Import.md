# Import

Solidity에서 코드를 재사용하고 모듈화를 통해 계약을 더 쉽게 관리할 수 있도록 도와주는 기능 중 하나가 `import`이다.
`import`를 사용하면 다른 파일에서 정의된 코드, 구조체, 함수 등을 현재 파일로 불러와 사용할 수 있다.
`import`는 로컬 파일과 외부 파일 모두 지원한다.

## 로컬 파일 임포트

로컬 파일은 동일한 프로젝트 폴더 구조 내에서 파일을 불러오는 것이다.

### 예시


#### 폴더 구조

```
├── Import.sol
└── Foo.sol
```

#### Foo.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

struct Point {
    uint256 x;
    uint256 y;
}

error Unauthorized(address caller);

function add(uint256 x, uint256 y) pure returns (uint256) {
    return x + y;
}

contract Foo {
    string public name = "Foo";
}
```

#### Import.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

// 현재 디렉토리에서 Foo.sol을 임포트
import "./Foo.sol";

// 특정 심볼만 임포트하고 별칭(alias)을 사용할 수 있다
import {Unauthorized, add as func, Point} from "./Foo.sol";

contract Import {
    // Foo 계약 초기화
    Foo public foo = new Foo();

    // Foo 계약의 name 값을 가져오는 함수
    function getFooName() public view returns (string memory) {
        return foo.name();
    }
}
```

### 예시 설명
- `import "./Foo.sol";`: 현재 디렉토리에서 `Foo.sol` 파일 전체를 임포트한다.
- `import {Unauthorized, add as func, Point} from "./Foo.sol";`: `Foo.sol`에서 특정 심볼(여기서는 `Unauthorized` 에러, `add`
  함수, `Point` 구조체)만 임포트하며, `add` 함수는 `func`라는 별칭으로 사용한다.
- `Foo` 계약을 초기화하고 `name` 값을 가져오는 예제를 포함한다.

## 외부 파일 임포트

Solidity에서는 외부 URL, 특히 GitHub와 같은 원격 저장소에서 직접 파일을 임포트할 수 있다. 이는 공용 라이브러리나 계약을 재사용할 때 유용하다.

### 예시

```solidity
// https://github.com/owner/repo/blob/branch/path/to/Contract.sol
import "https://github.com/owner/repo/blob/branch/path/to/Contract.sol";

// OpenZeppelin 라이브러리의 ECDSA.sol 임포트 예시
// https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.5/contracts/utils/cryptography/ECDSA.sol
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.5/contracts/utils/cryptography/ECDSA.sol";
```

### 예시 설명
- `import "https://github.com/owner/repo/blob/branch/path/to/Contract.sol";`: GitHub 저장소에서 특정 파일을 임포트한다. URL은 파일의 정확한
  경로를 포함해야 한다.
- `import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.5/contracts/utils/cryptography/ECDSA.sol";`:
  OpenZeppelin의 `ECDSA.sol` 파일을 임포트하는 예시이다. 이는 보안 관련 유틸리티 함수들을 제공한다.





## 미션 : Solidity Import 실습

1. [Remix IDE](https://remix.ethereum.org/)에 접속한다.
2. 새 파일을 만들어 예제 코드를 입력한다.
3. 코드 작성 및 수정: 각 단계별로 주석을 풀거나 import 방법을 변경하며 코드를 수정한다.
4. 컴파일: 각 수정 후 코드를 컴파일하여 오류 발생 여부를 확인한다.
5. 배포: 수정된 코드를 배포하고 배포된 계약 인스턴스를 사용하여 함수를 호출한다.
6. 결과 확인: 함수 호출 결과를 확인하고 예상된 동작과 실제 동작을 비교한다.

### 단계별 미션 수행

#### 1. 로컬 파일 임포트

##### 1.1 `Foo.sol` 파일 작성

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

struct Point {
    uint256 x;
    uint256 y;
}

error Unauthorized(address caller);

function add(uint256 x, uint256 y) pure returns (uint256) {
    return x + y;
}

contract Foo {
    string public name = "Foo";
}
```

##### 1.2 `Import.sol` 파일 작성 및 테스트

- `Import.sol` 파일에 `Foo.sol`을 임포트
  ```solidity
  // SPDX-License-Identifier: MIT
  pragma solidity ^0.8.24;

  // import Foo.sol from current directory
  import "./Foo.sol";

  // import {symbol1 as alias, symbol2} from "filename";
  import {Unauthorized, add as func, Point} from "./Foo.sol";

  contract Import {
      // Initialize Foo.sol
      Foo public foo = new Foo();

      // Test Foo.sol by getting its name
      function getFooName() public view returns (string memory) {
          return foo.name();
      }
  }
  ```
  - `Import.sol` 파일을 작성하고 컴파일한다.
  - 배포 후 `getFooName` 함수를 호출하여 `Foo` 계약의 `name` 값을 확인한다.

### 2. 외부 파일 임포트

#### 2.1 OpenZeppelin 라이브러리 임포트 및 테스트

- `Import.sol` 파일에 외부 파일을 임포트
  ```solidity
  // SPDX-License-Identifier: MIT
  pragma solidity ^0.8.24;

  import "./Foo.sol";
  import {Unauthorized, add as func, Point} from "./Foo.sol";
  import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/release-v4.5/contracts/utils/cryptography/ECDSA.sol";

  contract Import {
      Foo public foo = new Foo();

      function getFooName() public view returns (string memory) {
          return foo.name();
      }

      function verifySignature(bytes32 hash, bytes memory signature, address signer) public pure returns (bool) {
          return ECDSA.recover(hash, signature) == signer;
      }
  }
  ```
  - `Import.sol` 파일을 작성하고 컴파일한다.
  - 배포 후 `getFooName` 함수를 호출하여 `Foo` 계약의 `name` 값을 확인한다.
  - `verifySignature` 함수를 호출하여 서명 검증 기능을 테스트한다.

### 3. 심볼 임포트 및 테스트

#### 3.1 특정 심볼 임포트 및 테스트

- `Import.sol` 파일에서 특정 심볼을 임포트
  ```solidity
  // SPDX-License-Identifier: MIT
  pragma solidity ^0.8.24;

  import {Unauthorized, add as func, Point} from "./Foo.sol";

  contract Import {
      Point public point;

      function setPoint(uint256 _x, uint256 _y) public {
          point = Point(_x, _y);
      }

      function addPoints(uint256 _x, uint256 _y) public pure returns (uint256) {
          return func(_x, _y);
      }

      function triggerError() public view {
          revert Unauthorized(msg.sender);
      }
  }
  ```
  - `Import.sol` 파일을 작성하고 컴파일한다.
  - 배포 후 `setPoint` 함수를 호출하여 `Point` 구조체 값을 설정한다.
  - `addPoints` 함수를 호출하여 두 숫자를 더한 값을 확인한다.
  - `triggerError` 함수를 호출하여 `Unauthorized` 에러가 발생하는지 확인한다.
