## Constructor란 무엇인가?

Constructor(생성자)는 Solidity에서 스마트 계약이 배포될 때 한 번 실행되는 특수 함수이다. Constructor는 계약의 초기 상태를 설정하는 데 사용된다. Constructor는 계약을 배포할 때 필요한 초기 매개변수를 받을 수 있으며, 이 매개변수를 통해 상태 변수를 초기화하거나 초기 설정 작업을 수행할 수 있다. 

## Constructor의 기본 구조

Solidity에서 생성자는 constructor 키워드를 사용하여 정의됩니다. 생성자의 가시성은 public 또는 internal일 수 있지만,  
internal로 정의된 constructor를 갖는 contract는 abstract contract로 기능한다. 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract MyContract {
    uint public value;

    // Constructor 
    constructor(uint _value) public {
        value = _value;
    }
}
```
### 구성 요소 설명 

구성 요소 설명
constructor : constructor 키워드를 통해 constructor가 선언됨을 알립니다.
매개변수 (Parameters) : 생성자가 입력으로 받을 값을 정의합니다. 각 매개변수는 타입과 함께 정의됩니다.
가시성 (Visibility) : 생성자의 접근 범위를 지정합니다. public, internal 두가지 경우가 가능합니다.  

constructor의 내부에는 컨트랙트가 실행될 때 상태 변수를 초기화 하거나, 필요한 초기 설정 작업을 위한 코드가 들어갑니다. 



## 예제 코드

예제를 통해 constructor를 이해해보자. 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract MyContract {
    uint public value;
    address public owner;

    // 생성자
    constructor(uint _value) {
        value = _value;
        owner = msg.sender;
    }

    // 상태 변수 반환 함수
    function getValue() public view returns (uint) {
        return value;
    }

    function getOwner() public view returns (address) {
        return owner;
    }
}
```

#### constructor 정의

```solidity
    constructor(uint _value) {
        value = _value;
        owner = msg.sender;
    }
```

constructor를 정의합니다. 매개변수로 _value 값을 받아 상태 변수 value와 owner를 각각 초기화합니다
owner을 constructor에서 초기화 하는 것은 많은 컨트랙트들이 채택하는 방식입니다. 

### getValue 함수

```solidity
    function getValue() public view returns (uint) {
            return value;
        }
```

view function으로 value값을 리턴합니다. 

### getOwner 함수

```solidity
    function getOwner() public view returns (address) {
        return owner;
    }
```
view function으로 owner값을 리턴합니다. 


## Remix에서 실습 
1. Remix에서 새로운 solidity 파일 생성해서 예제 코드를 복사 붙여넣기 합니다.
2. 예제 코드를 compile 후 deploy합니다.
3. 아래 버튼들이 제대로 동작하는지 확인합니다.

- value에 1을 넣고 deploy 합니다.
  
![image](https://github.com/mmingyeomm/nestJS/assets/87323564/66da8dce-47ac-4370-a30e-17276fc776d4)


- getValue와 getOwner을 통해 value와 owner가 각각 1, msg.sender로 초기화 되었음을 확인합니다. 
![image](https://github.com/mmingyeomm/nestJS/assets/87323564/7f5ca5fb-a66c-4e06-8f30-743164a9468c)
