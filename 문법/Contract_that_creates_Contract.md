# 컨트랙트를 생성하는 컨트랙트  

솔리디티는 스마트 컨트랙트를 통해 다른 컨트랙트를 생성할 수 있습니다. 


예제 코드를 통한 이해가 가장 직관적이니 바로 실습으로 들어가 이해해보도록 합시다. 

## 예제 코드 

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;


contract Car {
    address public owner;
    string public model;
    address public carAddr;

    constructor(address _owner, string memory _model) payable {
        owner = _owner;
        model = _model;
        carAddr = address(this);
    }
}

contract CarFactory {
    Car[] public cars;

    function create(address _owner, string memory _model) public {
        Car car = new Car(_owner, _model);
        cars.push(car);
    }

    function createAndSendEther(address _owner, string memory _model)
        public
        payable
    {
        Car car = (new Car){value: msg.value}(_owner, _model);
        cars.push(car);
    }

    function getCar(uint256 _index)
        public
        view
        returns (
            address owner,
            string memory model,
            address carAddr,
            uint256 balance
        )
    {
        Car car = cars[_index];

        return (car.owner(), car.model(), car.carAddr(), address(car).balance);
    }
}
```

###  Car 컨트랙트 생성  

생성자로 owner와 model을 인자로 받는 Car 컨트랙트를 생성해줍니다.  

### CarFactory 컨트랙트 생성  

CarFactory 컨트랙트에서 Car 컨트랙트를 생성하고 생성된 컨트랙트들을 cars 변수에 저장할 수 있습니다.  

### create 함수  

CarFactory 컨트랙트에서 new 키워드를 통해 car을 생성하고 cars에 이를 푸쉬합니다.  

### createAndSendEther 함수  

컨트랙트를 **생성** 하는 것이기 때문에 컨트랙트에 송금하는 것도 가능합니다.  

### getCar 함수  

getCar 함수를 통해 생성된 컨트랙트의 owner, model, carAddr, balance를 조회할 수 있습니다. 

## Remix에서 실습

1. Remix에서 새로운 solidity 파일 생성해서 예제 코드를 복사 붙여넣기 합니다.
2. 예제 코드를 compile 후 deploy합니다.
3. 아래 버튼들이 제대로 동작하는지 확인합니다.  
4. **아래 이미지와 같이 배포된 컨트랙트 중, CarFactory를 선택합니다.** 

![image](https://github.com/mmingyeomm/solidity_basic_module-/assets/87323564/8f2e8810-a56d-493a-a86a-e77830e40c66)


### 1. Create 

![image](https://github.com/mmingyeomm/solidity_basic_module-/assets/87323564/80bad851-b2e9-4205-87f1-3fe35848563a)

create 함수에 각각 owner 주소, mercedes 를 인자로 입력하여 Car을 생성합니다.  

### 2. createAndSend 

![image](https://github.com/mmingyeomm/solidity_basic_module-/assets/87323564/d30d3c83-0145-4fb2-bb62-f2a026306f16)

createAndSend 함수에 각각 owner 주소, volvo를 인자로 입력하고, 10wei를 송금 금액으로 설정 후, Car을 생성합니다.

### 3. getCar

getCar 에 각각 0과 1을 넣어보며 cars 변수에 내가 생성한 컨트랙트의 정보를 확인합니다. 

![image](https://github.com/mmingyeomm/solidity_basic_module-/assets/87323564/a8d46f20-1d98-474e-baab-9e03f87caf74)
![image](https://github.com/mmingyeomm/solidity_basic_module-/assets/87323564/c1f7762b-b0a3-4d20-af41-d7a0264ae0f5)

carAddr를 통해 내가 생성한 컨트랙트의 주소 또한 확인할 수 있습니다. 




