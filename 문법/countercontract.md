# Counting Contract

### Contract
Contract 함수는 Smart Contract 내에서 특정 작업을 수행하는 코드 블록입니다. 블록체인 네트워크에서 호출될 때 실행됩니다.<br>
</br>

#### Contract 실행 범위 
- public : 모든 Smart Contract가 호출할 수 있는 함수입니다. </br>
- private : 동일한 Smart Contract 내부에서만 호출될 수 있습니다. 상속받은 Smart Contract에서는 접근할 수 없습니다. </br>
- external : 외부 Smart Contract, 트랜젝션에 의해서만 호출될 수 있는 함수입니다. </br>
- internal : Smart Contract 내부에서만 호출될 수 있는 함수입니다. </br>

#### Contract 상태 변경 
- 상태를 변경하는 함수 : 블록체인 상태를 변경하며, 가스를 소비합니다.
- 조회 함수 : 상태를 변경하지 않으며, 가스를 소비하지 않습니다. 'view', 'pure' 키워드를 사용합니다.

## 예제 코드
Counting Contract, uint256 변수 조회, 증가, 감소 함수를 구현한 코드입니다.
```bash
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract Counter {
    // unsigned / 256-bit (32bytes) / Fixed size 256bits
    uint256 public count;

    // 현재 count 값을 get하는 함수 
    function get() public view returns (uint256) {
        return count;
    }

    // count 1씩 증가하는 함수 
    function inc() public {
        count += 1;
    }

    // count 1씩 감소하는 함수
    function dec() public {
        // count > 0 인 경우 실행, count <= 0, default 값 "Counter is already zero" 출력
        require(count > 0, "Counter is already zero");
        count -= 1;
    }
} 
```

## Remix에서 실습 
1. Remix에서 새로운 solidity 파일 생성해서 예제 코드를 복사 붙여넣기 합니다.
2. 예제 코드를 compile 후 deploy합니다.
3. 아래 버튼들이 제대로 동작하는지 확인합니다.

- inc를 누르고 count를 조회한 경우 <br>
<img src= "https://github.com/Joon2000/Solidity-modules/blob/2df6a8bb21bf2699e53bd30dfda121710522eb74/images/countercontract/increase.png" width="250px" height="400px" 
  title="increase" alt="increase"><br/>
- dec를 누르고 count를 조회한 경우 <br>
<img src= "https://github.com/Joon2000/Solidity-modules/blob/2df6a8bb21bf2699e53bd30dfda121710522eb74/images/countercontract/decrease.png" width="250px" height="400px" 
  title="decrease" alt="decrease"><br/>

- count가 0인 상태에서 dec를 진행한 경우<br>
<img src= "https://github.com/Joon2000/Solidity-modules/blob/2df6a8bb21bf2699e53bd30dfda121710522eb74/images/countercontract/deccount0.png" width="1000px" height="100px" 
  title="decrease0" alt="decrease0"><br/>
