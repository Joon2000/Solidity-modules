# State Variable 
Solidity에서 State Variable(State 변수)는 Smart Contract의 상태를 저장하는 데 사용되는 변수입니다. 
이 변수들은 블록체인에 영구적으로 저장되며, 계약의 상태를 나타내는 중요한 요소입니다. 

### State Variable의 특징 
- 영구 저장 : State Variable는 블록체인에 저장되기 때문에, 트랜잭션이 종료된 후에도 그 값이 유지됩니다. <br>
- 가스 비용 : State Variable를 작성하는 것은 가스 비용을 수반합니다. <br>
- 전역 접근 : State Variable는 Contract의 모든 함수에서 접근할 수 있습니다. Contract의 다양한 함수들이 같은 State를 공유합니다. <br>



## 예제 코드
Step-by-step instructions on how to install the project.
```bash
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24; 

contract SimpleStorage {
    // num값을 저장하기 위한 State 변수 선언
    uint256 public num;

    // State 변수의 값을 설정하는 경우, 트랜젝션을 보냅니다.
    function set(uint256 _num) public {
        num = _num;
    }

    // State 변수의 값을 조회하는 경우, 트랜잭션을 보내지 않습니다. 
    function get() public view returns (uint256) {
        return num;
    }
}

```


## Remix에서 실습 
1. Remix에서 새로운 solidity 파일 생성해서 예제 코드를 복사 붙여넣기 합니다.
2. 예제 코드를 compile 후 deploy합니다.
3. 아래 버튼들이 제대로 동작하는지 확인합니다.

- StateVariable이 작성된 경우
<img src= "https://github.com/Joon2000/Solidity-modules/blob/c4761d107c2dbf02f7c9680e619d87b1263cc26c/images/statevariable/SetStateVariable.png" width="1000px" height="400px" 
  title="setStateVariable" alt="setStateVariable"><br/>

- StateVariable 조회하는 경우 (트랜잭션 발생, 가스 비용 발생) <br>
<img src= "https://github.com/Joon2000/Solidity-modules/blob/c4761d107c2dbf02f7c9680e619d87b1263cc26c/images/statevariable/GetStateVariable.png" width="250px" height="400px" 
  title="getStateVariable" alt="getStateVariable"><br/>


