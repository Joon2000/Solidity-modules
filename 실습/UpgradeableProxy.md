# Proxy Contract

#### Proxy Contract를 이용한 Upgradable Contract 제작 실습

## 목표
1. NFT 판매자가 이 계약을 배포합니다.
2. 경매는 7일 동안 진행됩니다.
3. 참가자들은 현재 최고 입찰자보다 많은 ETH를 입금함으로써 입찰할 수 있습니다.
4. 최고 입찰자가 아닌 모든 입찰자는 자신의 입찰금을 철회할 수 있습니다.
5. 옥션 종료 후 최고 입찰자가 NFT의 새로운 소유자가 됩니다.
6. 판매자는 최고 입찰가의 ETH를 받습니다.

## Remix 세팅

### 1. Remix 접속
> Remix: <https://remix.ethereum.org/>
### 2. Workspace 생성
> 1. 왼쪽 상단의 workspaces 클릭 후 Create 클릭 <br><br>
> <img src="https://github.com/Joon2000/Solidity-modules/blob/main/images/english_auction/workspaces.png" width="250" height="400"></img><br><br><br>
> 2. Choose a template을 Blank 선택 Workspace name에 workspace 이름 적기<br><br>
> <img src="https://github.com/Joon2000/Solidity-modules/blob/main/images/english_auction/Create%20workspace.png" width="500" height="300"></img><br><br><br>
3. 모든 폴더와 파일 삭제<br><br>
4. contracts 폴더와 solidity 파일 생성<br><br>
> <img src="https://github.com/Joon2000/Solidity-modules/blob/main/images/english_auction/%E1%84%85%E1%85%A9%E1%86%AF%E1%84%83%E1%85%A5%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC.png" width="250" height="400"></img><br><br><br>

## 미션 진행

### 1. lisence와 solidity 컴파일 버전 설정
> lisence: MIT<br>
> 컴파일 버전: ^0.8.24<br>
 ```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;
```

### 2. Counter Contract 생성 (V1, V2)
> ConterV1 생성
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;
contract CounterV1 {
    uint256 public count;

    function inc() external {
        count += 1;
    }
}
```
> ConterV2 생성
```solidity
contract CounterV2 {
    uint256 public count;

    function inc() external {
        count += 1;
    }

    function dec() external {
        count -= 1;
    }
}

```

### Buggy Proxy Contract 생성

> 1. implementation, admin 변수 선언.  
각각 implementaion이 담겨있는 컨트랙트 주소와, implementation을 바꿀 권한이 있는 사람의 주소를 담고 있다. 
> 2. _delegate() 함수를 통해 implementation이 가리키는 contract에 **delegatecall**(delegatecall 부분 참고)을 수행.
> 3. upgradeTo() admin인 사람이 컨트랙트 호출 통해, implementation 주소를 바꿀 수 있게 한다. 


```solidity

contract BuggyProxy {
    address public implementation;
    address public admin;

    constructor() {
        admin = msg.sender;
    }

    function _delegate() private {
        (bool ok,) = implementation.delegatecall(msg.data);
        require(ok, "delegatecall failed");
    }

    fallback() external payable {
        _delegate();
    }

    receive() external payable {
        _delegate();
    }

    function upgradeTo(address _implementation) external {
        require(msg.sender == admin, "not authorized");
        implementation = _implementation;
    }
}
```

> Buggy Proxy인 이유는 뒤에 소개될 Proxy 컨트랙트에 비해 버그들에 더욱 노출되어 있기 때문이다. 앞으로 나올 Proxy Contract와의 비교를 통해 Contract에서 발생할 수 있는 버그에 대해서 생각해보자. 

### Proxy Contract 생성

> Proxy 컨트랙트는 BuggyProxy와 같은 역할을 수행하지만 구현을 달리하였다. 
>> 1. implementation, admin 같은 중요한 주소는 StorageSlot library를 이용하여, collision이 나지 않게 StorageSlot에   위치를 분배하여 저장하였다. 이를 통해 Collision에서 올 수 있는 버그를 방지한다. 
>> 2. modifier ifAdmin()을 통해 Admin만 중요 정보를 확인할 수 있게하여 노출을 최소화한다.
>> 3. Delegation함수를 inline assembly로 구현하여, 데이터를 효율적으로 관리한다. 



```solidity
contract Proxy {

    // 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc
    bytes32 private constant IMPLEMENTATION_SLOT =
        bytes32(uint256(keccak256("eip1967.proxy.implementation")) - 1);
    // 0xb53127684a568b3173ae13b9f8a6016e243e63b6e8ee1178d6a717850b5d6103
    bytes32 private constant ADMIN_SLOT =
        bytes32(uint256(keccak256("eip1967.proxy.admin")) - 1);

    constructor() {
        _setAdmin(msg.sender);
    }

    modifier ifAdmin() {
        if (msg.sender == _getAdmin()) {
            _;
        } else {
            _fallback();
        }
    }

    function _getAdmin() private view returns (address) {
        return StorageSlot.getAddressSlot(ADMIN_SLOT).value;
    }

    function _setAdmin(address _admin) private {
        require(_admin != address(0), "admin = zero address");
        StorageSlot.getAddressSlot(ADMIN_SLOT).value = _admin;
    }

    function _getImplementation() private view returns (address) {
        return StorageSlot.getAddressSlot(IMPLEMENTATION_SLOT).value;
    }

    function _setImplementation(address _implementation) private {
        require(
            _implementation.code.length > 0, "implementation is not contract"
        );
        StorageSlot.getAddressSlot(IMPLEMENTATION_SLOT).value = _implementation;
    }

    function changeAdmin(address _admin) external ifAdmin {
        _setAdmin(_admin);
    }

    function upgradeTo(address _implementation) external ifAdmin {
        _setImplementation(_implementation);
    }

    function admin() external ifAdmin returns (address) {
        return _getAdmin();
    }

    function implementation() external ifAdmin returns (address) {
        return _getImplementation();
    }

    function _delegate(address _implementation) internal virtual {
        assembly {
            calldatacopy(0, 0, calldatasize())

            let result :=
                delegatecall(gas(), _implementation, 0, calldatasize(), 0, 0)
          
            returndatacopy(0, 0, returndatasize())

            switch result
            case 0 {
                revert(0, returndatasize())
            }
            default {
                return(0, returndatasize())
            }
        }
    }

    function _fallback() private {
        _delegate(_getImplementation());
    }

    fallback() external payable {
        _fallback();
    }

    receive() external payable {
        _fallback();
    }
}

``` 
### Proxy Contract 생성

> Proxy 컨트랙트는 BuggyProxy와 같은 역할을 수행하지만 구현을 달리하였다. 
>> 1. implementation, admin 같은 중요한 주소는 StorageSlot library를 이용하여, collision이 나지 않게 StorageSlot에   위치를 분배하여 저장하였다. 이를 통해 Collision에서 올 수 있는 버그를 방지한다. 
>> 2. modifier ifAdmin()을 통해 Admin만 중요 정보를 확인할 수 있게하여 노출을 최소화한다.
>> 3. Delegation함수를 inline assembly로 구현하여, 데이터를 효율적으로 관리한다. 


```solidity
contract Proxy {

    // 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc
    bytes32 private constant IMPLEMENTATION_SLOT =
        bytes32(uint256(keccak256("eip1967.proxy.implementation")) - 1);
    // 0xb53127684a568b3173ae13b9f8a6016e243e63b6e8ee1178d6a717850b5d6103
    bytes32 private constant ADMIN_SLOT =
        bytes32(uint256(keccak256("eip1967.proxy.admin")) - 1);

    constructor() {
        _setAdmin(msg.sender);
    }

    modifier ifAdmin() {
        if (msg.sender == _getAdmin()) {
            _;
        } else {
            _fallback();
        }
    }

    function _getAdmin() private view returns (address) {
        return StorageSlot.getAddressSlot(ADMIN_SLOT).value;
    }

    function _setAdmin(address _admin) private {
        require(_admin != address(0), "admin = zero address");
        StorageSlot.getAddressSlot(ADMIN_SLOT).value = _admin;
    }

    function _getImplementation() private view returns (address) {
        return StorageSlot.getAddressSlot(IMPLEMENTATION_SLOT).value;
    }

    function _setImplementation(address _implementation) private {
        require(
            _implementation.code.length > 0, "implementation is not contract"
        );
        StorageSlot.getAddressSlot(IMPLEMENTATION_SLOT).value = _implementation;
    }

    function changeAdmin(address _admin) external ifAdmin {
        _setAdmin(_admin);
    }

    function upgradeTo(address _implementation) external ifAdmin {
        _setImplementation(_implementation);
    }

    function admin() external ifAdmin returns (address) {
        return _getAdmin();
    }

    function implementation() external ifAdmin returns (address) {
        return _getImplementation();
    }

    function _delegate(address _implementation) internal virtual {
        assembly {
            calldatacopy(0, 0, calldatasize())

            let result :=
                delegatecall(gas(), _implementation, 0, calldatasize(), 0, 0)
          
            returndatacopy(0, 0, returndatasize())

            switch result
            case 0 {
                revert(0, returndatasize())
            }
            default {
                return(0, returndatasize())
            }
        }
    }

    function _fallback() private {
        _delegate(_getImplementation());
    }

    fallback() external payable {
        _fallback();
    }

    receive() external payable {
        _fallback();
    }
}

``` 


### 3. ProxyAdmin Contract 생성

> ProxyAdmin Contract를 통해 proxy contract 내부의 함수를 호출한다. 
>> 1. implementation, admin 같은 중요한 주소는 StorageSlot library를 이용하여, collision이 나지 않게 StorageSlot에   위치를 분배하여 저장하였다. 이를 통해 Collision에서 올 수 있는 버그를 방지한다. 
>> 2. modifier ifAdmin()을 통해 Admin만 중요 정보를 확인할 수 있게하여 노출을 최소화한다.
>> 3. Delegation함수를 inline assembly로 구현하여, 데이터를 효율적으로 관리한다. 


```solidity
contract ProxyAdmin {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "not owner");
        _;
    }

    function getProxyAdmin(address proxy) external view returns (address) {
        (bool ok, bytes memory res) =
            proxy.staticcall(abi.encodeCall(Proxy.admin, ()));
        require(ok, "call failed");
        return abi.decode(res, (address));
    }

    function getProxyImplementation(address proxy)
        external
        view
        returns (address)
    {
        (bool ok, bytes memory res) =
            proxy.staticcall(abi.encodeCall(Proxy.implementation, ()));
        require(ok, "call failed");
        return abi.decode(res, (address));
    }

    function changeProxyAdmin(address payable proxy, address admin)
        external
        onlyOwner
    {
        Proxy(proxy).changeAdmin(admin);
    }

    function upgrade(address payable proxy, address implementation)
        external
        onlyOwner
    {
        Proxy(proxy).upgradeTo(implementation);
    }
}
``` 




### 4. StorageSlot library 생성
> StorageSlot library에서 특정 slot에 있는 정보를 가져올 수 있도록 한다.
>> 1. AddressSlot struct를 선언하여 address를 변수로 갖게 하다.
>> 2. getAddressSlot()function은 slot을 인자로 받아 해당 주소에 있는 값을 return한다. 
```solidity
library StorageSlot {
    struct AddressSlot {
        address value;
    }

    function getAddressSlot(bytes32 slot)
        internal
        pure
        returns (AddressSlot storage r)
    {
        assembly {
            r.slot := slot
        }
    }
}
```

### 5. TestSlot contract 생성
> StorageSlot library가 잘 작동하는지 TestSlot을 통해 확인한다.
>> 1. AddressSlot struct를 선언하여 address를 변수로 갖게 하다.
>> 2. slot 변수를 선언해 keccak256 방식으로 인코딩한 데이터를 저장한다.
>> 3. getSlot(slot) slot변수의 위치에 있는 slot에서
>> 4. writeSlot()으로 slot변수의 위치에 있는 slot에서 address값을 저장한다. 

```solidity
contract TestSlot {
    bytes32 public constant slot = keccak256("TEST_SLOT");

    function getSlot() external view returns (address) {
        return StorageSlot.getAddressSlot(slot).value;
    }

    function writeSlot(address _addr) external {
        StorageSlot.getAddressSlot(slot).value = _addr;
    }
}

```

## 전체 코드
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract CounterV1 {
    uint256 public count;

    function inc() external {
        count += 1;
    }
}

contract CounterV2 {
    uint256 public count;

    function inc() external {
        count += 1;
    }

    function dec() external {
        count -= 1;
    }
}

contract BuggyProxy {
    address public implementation;
    address public admin;

    constructor() {
        admin = msg.sender;
    }

    function _delegate() private {
        (bool ok,) = implementation.delegatecall(msg.data);
        require(ok, "delegatecall failed");
    }

    fallback() external payable {
        _delegate();
    }

    receive() external payable {
        _delegate();
    }

    function upgradeTo(address _implementation) external {
        require(msg.sender == admin, "not authorized");
        implementation = _implementation;
    }
}

contract Dev {
    function selectors() external view returns (bytes4, bytes4, bytes4) {
        return (
            Proxy.admin.selector,
            Proxy.implementation.selector,
            Proxy.upgradeTo.selector
        );
    }
}

contract Proxy {

    // 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc
    bytes32 private constant IMPLEMENTATION_SLOT =
        bytes32(uint256(keccak256("eip1967.proxy.implementation")) - 1);
    // 0xb53127684a568b3173ae13b9f8a6016e243e63b6e8ee1178d6a717850b5d6103
    bytes32 private constant ADMIN_SLOT =
        bytes32(uint256(keccak256("eip1967.proxy.admin")) - 1);

    constructor() {
        _setAdmin(msg.sender);
    }

    modifier ifAdmin() {
        if (msg.sender == _getAdmin()) {
            _;
        } else {
            _fallback();
        }
    }

    function _getAdmin() private view returns (address) {
        return StorageSlot.getAddressSlot(ADMIN_SLOT).value;
    }

    function _setAdmin(address _admin) private {
        require(_admin != address(0), "admin = zero address");
        StorageSlot.getAddressSlot(ADMIN_SLOT).value = _admin;
    }

    function _getImplementation() private view returns (address) {
        return StorageSlot.getAddressSlot(IMPLEMENTATION_SLOT).value;
    }

    function _setImplementation(address _implementation) private {
        require(
            _implementation.code.length > 0, "implementation is not contract"
        );
        StorageSlot.getAddressSlot(IMPLEMENTATION_SLOT).value = _implementation;
    }

    function changeAdmin(address _admin) external ifAdmin {
        _setAdmin(_admin);
    }

    function upgradeTo(address _implementation) external ifAdmin {
        _setImplementation(_implementation);
    }

    function admin() external ifAdmin returns (address) {
        return _getAdmin();
    }

    function implementation() external ifAdmin returns (address) {
        return _getImplementation();
    }

    function _delegate(address _implementation) internal virtual {
        assembly {
           
            calldatacopy(0, 0, calldatasize())

            let result :=
                delegatecall(gas(), _implementation, 0, calldatasize(), 0, 0)

            returndatacopy(0, 0, returndatasize())

            switch result
            case 0 {
                revert(0, returndatasize())
            }
            default {
                return(0, returndatasize())
            }
        }
    }

    function _fallback() private {
        _delegate(_getImplementation());
    }

    fallback() external payable {
        _fallback();
    }

    receive() external payable {
        _fallback();
    }
}

contract ProxyAdmin {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "not owner");
        _;
    }

    function getProxyAdmin(address proxy) external view returns (address) {
        (bool ok, bytes memory res) =
            proxy.staticcall(abi.encodeCall(Proxy.admin, ()));
        require(ok, "call failed");
        return abi.decode(res, (address));
    }


    function getProxyImplementation(address proxy)
        external
        view
        returns (address)
    {
        (bool ok, bytes memory res) =
            proxy.staticcall(abi.encodeCall(Proxy.implementation, ()));
        require(ok, "call failed");
        return abi.decode(res, (address));
    }

    function changeProxyAdmin(address payable proxy, address admin)
        external
        onlyOwner
    {
        Proxy(proxy).changeAdmin(admin);
    }

    function upgrade(address payable proxy, address implementation)
        external
        onlyOwner
    {
        Proxy(proxy).upgradeTo(implementation);
    }
}

library StorageSlot {
    struct AddressSlot {
        address value;
    }

    function getAddressSlot(bytes32 slot)
        internal
        pure
        returns (AddressSlot storage r)
    {
        assembly {
            r.slot := slot
        }
    }
}

contract TestSlot {
    bytes32 public constant slot = keccak256("TEST_SLOT");

    function getSlot() external view returns (address) {
        return StorageSlot.getAddressSlot(slot).value;
    }

    function writeSlot(address _addr) external {
        StorageSlot.getAddressSlot(slot).value = _addr;
    }
}

```
#### Open in [REMIX](https://remix.ethereum.org/?#code=Ly8gU1BEWC1MaWNlbnNlLUlkZW50aWZpZXI6IE1JVApwcmFnbWEgc29saWRpdHkgXjAuOC4yNDsKCmludGVyZmFjZSBJRVJDNzIxIHsKICAgIGZ1bmN0aW9uIHNhZmVUcmFuc2ZlckZyb20oYWRkcmVzcyBmcm9tLCBhZGRyZXNzIHRvLCB1aW50MjU2IHRva2VuSWQpCiAgICAgICAgZXh0ZXJuYWw7CiAgICBmdW5jdGlvbiB0cmFuc2ZlckZyb20oYWRkcmVzcywgYWRkcmVzcywgdWludDI1NikgZXh0ZXJuYWw7Cn0KCmNvbnRyYWN0IEVuZ2xpc2hBdWN0aW9uIHsKICAgIGV2ZW50IFN0YXJ0KCk7CiAgICBldmVudCBCaWQoYWRkcmVzcyBpbmRleGVkIHNlbmRlciwgdWludDI1NiBhbW91bnQpOwogICAgZXZlbnQgV2l0aGRyYXcoYWRkcmVzcyBpbmRleGVkIGJpZGRlciwgdWludDI1NiBhbW91bnQpOwogICAgZXZlbnQgRW5kKGFkZHJlc3Mgd2lubmVyLCB1aW50MjU2IGFtb3VudCk7CgogICAgSUVSQzcyMSBwdWJsaWMgbmZ0OwogICAgdWludDI1NiBwdWJsaWMgbmZ0SWQ7CgogICAgYWRkcmVzcyBwYXlhYmxlIHB1YmxpYyBzZWxsZXI7CiAgICB1aW50MjU2IHB1YmxpYyBlbmRBdDsKICAgIGJvb2wgcHVibGljIHN0YXJ0ZWQ7CiAgICBib29sIHB1YmxpYyBlbmRlZDsKCiAgICBhZGRyZXNzIHB1YmxpYyBoaWdoZXN0QmlkZGVyOwogICAgdWludDI1NiBwdWJsaWMgaGlnaGVzdEJpZDsKICAgIG1hcHBpbmcoYWRkcmVzcyA9PiB1aW50MjU2KSBwdWJsaWMgYmlkczsKCiAgICBjb25zdHJ1Y3RvcihhZGRyZXNzIF9uZnQsIHVpbnQyNTYgX25mdElkLCB1aW50MjU2IF9zdGFydGluZ0JpZCkgewogICAgICAgIG5mdCA9IElFUkM3MjEoX25mdCk7CiAgICAgICAgbmZ0SWQgPSBfbmZ0SWQ7CgogICAgICAgIHNlbGxlciA9IHBheWFibGUobXNnLnNlbmRlcik7CiAgICAgICAgaGlnaGVzdEJpZCA9IF9zdGFydGluZ0JpZDsKICAgIH0KCiAgICBmdW5jdGlvbiBzdGFydCgpIGV4dGVybmFsIHsKICAgICAgICByZXF1aXJlKCFzdGFydGVkLCAic3RhcnRlZCIpOwogICAgICAgIHJlcXVpcmUobXNnLnNlbmRlciA9PSBzZWxsZXIsICJub3Qgc2VsbGVyIik7CgogICAgICAgIG5mdC50cmFuc2ZlckZyb20obXNnLnNlbmRlciwgYWRkcmVzcyh0aGlzKSwgbmZ0SWQpOwogICAgICAgIHN0YXJ0ZWQgPSB0cnVlOwogICAgICAgIGVuZEF0ID0gYmxvY2sudGltZXN0YW1wICsgNyBkYXlzOwoKICAgICAgICBlbWl0IFN0YXJ0KCk7CiAgICB9CgogICAgZnVuY3Rpb24gYmlkKCkgZXh0ZXJuYWwgcGF5YWJsZSB7CiAgICAgICAgcmVxdWlyZShzdGFydGVkLCAibm90IHN0YXJ0ZWQiKTsKICAgICAgICByZXF1aXJlKGJsb2NrLnRpbWVzdGFtcCA8IGVuZEF0LCAiZW5kZWQiKTsKICAgICAgICByZXF1aXJlKG1zZy52YWx1ZSA+IGhpZ2hlc3RCaWQsICJ2YWx1ZSA8IGhpZ2hlc3QiKTsKCiAgICAgICAgaWYgKGhpZ2hlc3RCaWRkZXIgIT0gYWRkcmVzcygwKSkgewogICAgICAgICAgICBiaWRzW2hpZ2hlc3RCaWRkZXJdICs9IGhpZ2hlc3RCaWQ7CiAgICAgICAgfQoKICAgICAgICBoaWdoZXN0QmlkZGVyID0gbXNnLnNlbmRlcjsKICAgICAgICBoaWdoZXN0QmlkID0gbXNnLnZhbHVlOwoKICAgICAgICBlbWl0IEJpZChtc2cuc2VuZGVyLCBtc2cudmFsdWUpOwogICAgfQoKICAgIGZ1bmN0aW9uIHdpdGhkcmF3KCkgZXh0ZXJuYWwgewogICAgICAgIHVpbnQyNTYgYmFsID0gYmlkc1ttc2cuc2VuZGVyXTsKICAgICAgICBiaWRzW21zZy5zZW5kZXJdID0gMDsKICAgICAgICBwYXlhYmxlKG1zZy5zZW5kZXIpLnRyYW5zZmVyKGJhbCk7CgogICAgICAgIGVtaXQgV2l0aGRyYXcobXNnLnNlbmRlciwgYmFsKTsKICAgIH0KCiAgICBmdW5jdGlvbiBlbmQoKSBleHRlcm5hbCB7CiAgICAgICAgcmVxdWlyZShzdGFydGVkLCAibm90IHN0YXJ0ZWQiKTsKICAgICAgICByZXF1aXJlKGJsb2NrLnRpbWVzdGFtcCA+PSBlbmRBdCwgIm5vdCBlbmRlZCIpOwogICAgICAgIHJlcXVpcmUoIWVuZGVkLCAiZW5kZWQiKTsKCiAgICAgICAgZW5kZWQgPSB0cnVlOwogICAgICAgIGlmIChoaWdoZXN0QmlkZGVyICE9IGFkZHJlc3MoMCkpIHsKICAgICAgICAgICAgbmZ0LnNhZmVUcmFuc2ZlckZyb20oYWRkcmVzcyh0aGlzKSwgaGlnaGVzdEJpZGRlciwgbmZ0SWQpOwogICAgICAgICAgICBzZWxsZXIudHJhbnNmZXIoaGlnaGVzdEJpZCk7CiAgICAgICAgfSBlbHNlIHsKICAgICAgICAgICAgbmZ0LnNhZmVUcmFuc2ZlckZyb20oYWRkcmVzcyh0aGlzKSwgc2VsbGVyLCBuZnRJZCk7CiAgICAgICAgfQoKICAgICAgICBlbWl0IEVuZChoaWdoZXN0QmlkZGVyLCBoaWdoZXN0QmlkKTsKICAgIH0KfQo&lang=en&optimize=false&runs=200&evmVersion=null&version=soljson-v0.8.25+commit.b61c2a91.js)

## 실행

### 폴더 구조
아래와 같은 폴더 구조를 만들어야 합니다.  
![image](https://github.com/mmingyeomm/nestJS/assets/87323564/d485f711-84cc-4a9b-bacb-66e45e621240)  


### 컨트랙트 배포
![image](https://github.com/mmingyeomm/nestJS/assets/87323564/d485f711-84cc-4a9b-bacb-66e45e621240)

>1. CounterV1, CounterV2, Proxy, ProxyAdmin 을 차례로 배포한다.  
    배포할 컨트랙트는 Deploy 버튼 위에 CONTRACT 메뉴를 통해 선택할 수 있다.  

![image](https://github.com/mmingyeomm/nestJS/assets/87323564/f2998ba1-6fdb-4e96-a0b4-1287777b4ff5)

>2. Proxy 컨트랙트 선택 후, upgradeTo에 CounterV1컨트랙트의 주소를 입력한다.  

![image](https://github.com/mmingyeomm/nestJS/assets/87323564/d6a2aa01-521d-47b3-9d25-0be9428d9066)

>3. Implementation 버튼을 누르고 컨트랙트 log의 decoded output을 통해 CounterV1을 ProxyContract가 적용하는 것을 확인할 수 있다.  

![image](https://github.com/mmingyeomm/nestJS/assets/87323564/803d2ee6-da00-472f-8195-3aaefc1f8a71)

>4. Proxy 컨트랙트 선택 후, changeAdmin에 ProxyAdmin 컨트랙트 address를 입력한다.  

![image](https://github.com/mmingyeomm/nestJS/assets/87323564/f870c386-6d07-47b1-b586-5c9868f5af2c)

>5. ProxyAdmin 컨트랙트에 getProxyAdmin, getProxyImplementation에 Proxy 컨트랙트의 주소를 입력하여 결과를 확인한다.

![image](https://github.com/mmingyeomm/nestJS/assets/87323564/bfc45371-96b0-4b81-b4b4-16d30cb328b1)

>6. At Address에 CounterV1의 주소를 입력하여, Proxy 컨트랙트를 통해 CounterV1을 호출할 수 있다.  

![image](https://github.com/mmingyeomm/nestJS/assets/87323564/ed0ab94e-3a6f-427f-8e79-b117b711baca)

>7. inc버튼을 눌러 Proxy 컨트랙트를 통해 호출된 CounterV1 컨트랙트가 동작하는 것을 확인할 수 있다. inc 버튼을 누르고 count를 확인한다.  

![image](https://github.com/mmingyeomm/nestJS/assets/87323564/de493c23-e21f-4265-b657-ea44db8ce1d7)

>8. At Address에 V2의 주소를 입력하여, Proxy 컨트랙트를 통해 V2을 호출할 수 있다.  

![image](https://github.com/mmingyeomm/nestJS/assets/87323564/bd360506-4f26-4bad-b0f2-fe14facbf210)

>9. 호출된 V2 컨트랙트에서 dec 함수를 실행시키면 다음과 같이 오류가 나는 것을 확인할 수 있다.  

![image](https://github.com/mmingyeomm/nestJS/assets/87323564/c2b4ce67-150c-4207-b363-2c1e796ff6c6)  

    **Proxy 컨트랙트에서 CounterV1을 implementation으로 선택하였기 때문에, CounterV2의 dec가 실행이 안되는 것이다.**  

>10. ProxyAdmin에서 changeProxy에 CounterV2의 주소를 입력한다.  

![image](https://github.com/mmingyeomm/nestJS/assets/87323564/c8cf1390-f27d-4fce-b9c1-9250d9562683)  

>11. 아까와 마찬가지로 At Address에 Proxy 주소를 입력하여 Proxy 컨트랙트를 통해 CounterV2를 호출한다.
>12. CounterV2의 dec 함수가 잘 실행되는 것을 확인한다. 
 
![image](https://github.com/mmingyeomm/nestJS/assets/87323564/a7170496-dc70-4da3-ba11-47dc28939210)
