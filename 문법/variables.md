# Variables 
Solidity의 변수에는 3가지 종류가 있습니다  

##  지역 변수 (Local Variables)  

### 정의 
- 지역 변수는 함수 내부에 정의 된 변수입니다.

### 특징 
- 함수가 실행될 때 메모리에 할당되고, 함수 실행이 끝나면 사라집니다. 
- 블록체인에 저장되지 않으므로 가스 비용이 상대적으로 저렴합니다.
- 선언된 함수 내에서만 사용할 수 있습니다. 

## 상태 변수 (State Variables)  

### 정의  
- 상태 변수는 함수 외부에서 선언되는 변수입니다. 

### 특징 
- 계약의 저장소(storage)에 저장되며, 블록체인에 영구적으로 저장됩니다.
- 블록체인에 저장되므로 가스 비용이 많이 듭니다. 
- 외부에서 접근할 수 있으며, public, internal, private, external 등의 접근 제어자를 사용할 수 있습니다.

## 전역 변수 (Global Variables) 



## 예제 코드

```bash
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract Variables {
    // State variables are stored on the blockchain.
    string public text = "Hello";
    uint256 public num = 123;

    function doSomething() public {
        // Local variables are not saved to the blockchain.
        uint256 i = 456;

        // Here are some global variables
        uint256 timestamp = block.timestamp; // Current block timestamp
        address sender = msg.sender; // address of the caller
    }
}
```