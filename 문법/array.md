# Array
Solidity에서 Array는 동일한 유형의 데이터 집합을 저장하는 데 사용됩니다. Compile-time fixed size, dynamic size 두 가지 유형으로 나눌 수 있습니다. 

#### Fixed-size Array
- 고정된 크기를 가지며 선언 시 크기를 정의합니다.
- 크기가 고정되어 있어 나중에 변경할 수 없습니다. (push, pop 함수를 사용 못 합니다)

#### Dynamic Array
- 크기가 고정되지 않고 동적으로 변경될 수 있습니다.
- 요소를 추가하거나 제거할 수 있습니다. 

#### Array의 특성 
- 'storage', 'memory' 키워드를 사용하여 array의 저장 위치를 지정할 수 있습니다.
    - 'storage' : State variable로서 블록체인에 저장됩니다.
    - 'memory' : function 내에서 임시로 사용되며, function 호출이 끝나면 사라집니다. 

## 예제 코드
```bash
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract Array {
    // Array를 초기화하는 여러가지 방법입니다.
    uint256[] public arr;
    uint256[] public arr2 = [1, 2, 3];
    // 고정된 크기의 array는 모든 값이 0으로 초기화됩니다.
    uint256[10] public myFixedSizeArr;

    // i번째 index에 있는 array 값 반환하는 함수입니다.
    function get(uint256 i) public view returns (uint256) {
        return arr[i];
    }

    // Solidity에서는 array 전체를 반환할 수 있습니다.
    // Array의 길이가 굉장히 커질 수 있어서 사용하지 않는 것을 추천합니다.
    function getArr() public view returns (uint256[] memory) {
        return arr;
    }

    function push(uint256 i) public {
        // python에서 append와 비슷한 함수 입니다.
        // array 마지막에 값을 추가하고, array의 길이는 1증가합니다.
        arr.push(i);
    }

    function pop() public {
        // python에서 pop과 비슷한 함수입니다.
        // array의 마지막 값을 삭제하고, array의 길이는 1 감소합니다.
        arr.pop();
    }

    // array의 길이를 반환하는 함수입니다.
    function getLength() public view returns (uint256) {
        return arr.length;
    }

    function remove(uint256 index) public {
        // delete는 array 길이에 영향을 주지 않습니다.
        // 기존에 있었던 값으로 reset합니다, 이 경우에는 0으로 값이 바뀝니다.
        delete arr[index];
    }
}
```


## Remix에서 실습 
1. Remix에서 새로운 solidity 파일 생성해서 예제 코드를 복사 붙여넣기 합니다.
2. 예제 코드를 compile 후 deploy합니다.
3. 아래 버튼들이 제대로 동작하는지 확인합니다.

- 초기화된 상태에서 getArr, getLength 버튼을 누르면 빈 array와 length가 0인 것을 확인할 수 있습니다. </br>
<img src= "https://github.com/Joon2000/Solidity-modules/blob/90822fecd8422ffa4551a7fbe96e7dc2933fbe44/images/array/init.png" width="250px" height="400px" 
  title="init" alt="init"><br/>

- 임의의 값을 추가하고, getArr, getLength 버튼을 누르면 array에 추가되었고, length가 1 증가한 것을 확인할 수 있습니다.
<img src= "https://github.com/Joon2000/Solidity-modules/blob/90822fecd8422ffa4551a7fbe96e7dc2933fbe44/images/array/push.png" width="250px" height="400px" 
  title="push" alt="push"><br/>

- Remove(index)를 진행하면 해당 index의 값은 지워지지만, length는 그대로인 것을 확인할 수 있습니다. </br>
<img src= "https://github.com/Joon2000/Solidity-modules/blob/90822fecd8422ffa4551a7fbe96e7dc2933fbe44/images/array/remove.png" width="250px" height="400px" 
  title="setStateVariable" alt="setStateVariable"><br/>

- 또한 pop()을 활용해 마지막 index에 존재하는 값을 지우는 것을 확인할 수 있습니다. </br>
<img src= "https://github.com/Joon2000/Solidity-modules/blob/90822fecd8422ffa4551a7fbe96e7dc2933fbe44/images/array/befpop.png" width="250px" height="400px" 
  title="setStateVariable" alt="setStateVariable">
<img src= "https://github.com/Joon2000/Solidity-modules/blob/90822fecd8422ffa4551a7fbe96e7dc2933fbe44/images/array/afpop.png" width="250px" height="400px" 
  title="setStateVariable" alt="setStateVariable"><br/>
