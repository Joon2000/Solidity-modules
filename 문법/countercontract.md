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

#### Contract 설명
이 예제는 간단한 카운터를 구현한 스마트 계약입니다. `count`라는 `uint256` 타입의 변수를 조회, 증가, 감소시키는 기능을 포함하고 있습니다.

#### 상태 변수
```solidity
uint256 public count;
```
- `count`는 unsigned 256-bit 정수형 변수로, 카운터의 값을 저장합니다. 
- `public` 키워드를 사용하여 자동으로 getter 함수가 생성됩니다.

#### get 함수
```solidity
function get() public view returns (uint256) {
    return count;
}
```
- 현재 `count` 값을 반환하는 함수입니다.
- `view` 키워드를 사용하여 상태를 변경하지 않고 조회만 합니다.

#### inc 함수
```solidity
function inc() public {
    count += 1;
}
```
- `count` 값을 1 증가시키는 함수입니다.
- `public` 키워드를 사용하여 누구나 호출할 수 있습니다.

#### dec 함수
```solidity
function dec() public {
    require(count > 0, "Counter is already zero");
    count -= 1;
}
```
- `count` 값을 1 감소시키는 함수입니다.
- `require` 구문을 사용하여 `count`가 0보다 큰 경우에만 감소시킵니다.
- `count`가 0인 경우, "Counter is already zero" 에러 메시지를 출력합니다.


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
