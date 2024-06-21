# English Auction

#### NFT 옥션 제작 실습

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
> <img src="https://github.com/Joon2000/English-Auction-Solidity/blob/main/images/workspaces.png" width="250" height="400"></img><br><br><br>
> 2. Choose a template을 Blank 선택 Workspace name에 workspace 이름 적기<br><br>
> <img src="https://github.com/Joon2000/English-Auction-Solidity/blob/main/images/Create%20workspace.png" width="500" height="300"></img><br><br><br>
3. 모든 폴더와 파일 삭제<br><br>
4. contracts 폴더와 solidity 파일 생성<br><br>
> <img src="https://github.com/Joon2000/English-Auction-Solidity/blob/main/images/%E1%84%85%E1%85%A9%E1%86%AF%E1%84%83%E1%85%A5%20%E1%84%89%E1%85%A2%E1%86%BC%E1%84%89%E1%85%A5%E1%86%BC.png" width="250" height="400"></img><br><br><br>

## 미션 진행

### 1. lisence와 solidity 컴파일 버전 설정
> lisence: MIT<br>
> 컴파일 버전: ^0.8.24<br>
 ```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;
```

### 2. IERC-721 interface 
> IERC721의 safeTransferFrom과 transferFrom 함수를 사용할 것입니다.
```solidity
interface IERC721 {
    function safeTransferFrom(address from, address to, uint256 tokenId)
        external;
    function transferFrom(address, address, uint256) external;
}
```

### 3. contract 생성
> EnglishAuction contract 생성
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

interface IERC721 {
    function safeTransferFrom(address from, address to, uint256 tokenId)
        external;
    function transferFrom(address, address, uint256) external;
}

contract EnglishAuction {}
```

### 4. event 설정
> 4개의 event를 만들 것입니다.
> 1. Start(): 경매가 시작되었음을 알리는 이벤트.
> 2. Bid(address indexed sender, uint256 amount): 주소 sender에서 금액 amount으로 새로운 입찰이 이루어졌음을 알리는 이벤트.
> 3. Withdraw(address indexed bidder, uint256 amount): 주소 bidder가 입찰 금액 amount을 철회함을 알리는 이벤트.
> 4. End(address winner, uint256 amount): 경매가 종료됨을 알리는 이벤트. <br><br>
```solidity
event Start();
event Bid(address indexed sender, uint256 amount);
event Withdraw(address indexed bidder, uint256 amount);
event End(address winner, uint256 amount);
```

### 5. 전역변수 설정
> 1. NFT 관련 변수들<br>
> - IERC721 public nft<br>
>> IERC721 인터페이스 타입의 public 변수. 이 변수는 경매에 사용될 NFT의 인터페이스를 참조합니다. 이를 통해 컨트랙트는 NFT의 소유권 이전과 관련된 함수들을 호출할 수 있습니다.<br>
> - uint256 public nftId<br>
>> 경매에 사용될 NFT의 고유 식별자(ID)입니다. 이 ID는 특정 NFT를 구분하고 참조하는 데 사용됩니다.<br><br>
```solidity
IERC721 public nft;
uint256 public nftId;
```

> 2. 옥션 관련 변수들<br>
> - address payable public seller<br>
>> NFT 판매자의 주소를 저장하는 변수입니다. 이 주소는 이더리움 네트워크 상의 지불 가능한 주소 형태로, 경매 종료 후 최고 입찰 금액을 이 주소로 전송합니다.<br>
> - uint256 public endAt<br>
>> 경매 종료 시점을 나타내는 타임스탬프입니다. 이 시점 이후에는 더 이상 입찰을 받지 않습니다.<br>
> - bool public started<br>
>> 경매가 시작되었는지 여부를 나타내는 변수입니다. 경매 시작 시 true로 설정되고, 그 전까지는 false입니다.<br>
> - bool public ended<br>
>> 경매가 종료되었는지 여부를 나타내는 변수입니다. 경매 종료 시 true로 설정됩니다.<br>
> - address public highestBidder<br>
>> 현재 최고 입찰자의 주소를 저장하는 변수입니다.<br>
> - uint256 public highestBid<br>
>> 현재까지의 최고 입찰 금액을 저장하는 변수입니다.<br>
> - mapping(address => uint256) public bids<br>
>> 각 주소에 대해 입찰된 금액을 저장하는 매핑입니다. 이는 입찰자가 입찰한 금액을 관리하고, 나중에 최고 입찰자가 아닌 경우 금액을 회수할 수 있도록 합니다.<br><br>
```solidity
address payable public seller;
uint256 public endAt;
bool public started;
bool public ended;

address public highestBidder;
uint256 public highestBid;
mapping(address => uint256) public bids;address payable public seller;
uint256 public endAt;
bool public started;
bool public ended;

address public highestBidder;
uint256 public highestBid;
mapping(address => uint256) public bids;
```
<br>

### 6. constructor 생성
> - 매개변수:<br>
>> address _nft: 경매에 사용될 NFT를 대표하는 스마트 컨트랙트의 주소입니다.<br>
>> uint256 _nftId: 경매에 사용될 NFT의 고유 식별자(ID)입니다.<br>
>> uint256 _startingBid: 경매 시작 시의 최저 입찰 금액입니다.<br>
> - 기능:<br>
>> nft = IERC721(_nft): 주어진 주소 _nft에서 IERC721 인터페이스를 구현하는 컨트랙트를 참조하여, nft 변수에 할당합니다. 이를 통해 경매 컨트랙트는 해당 NFT에 대한 다양한 작업을 수행할 수 있습니다.<br>
>> nftId = _nftId: 입력받은 NFT의 ID를 nftId 변수에 저장합니다. 이 ID는 특정 NFT를 경매 컨트랙트에서 참조하고 관리하는 데 사용됩니다.<br>
>> seller = payable(msg.sender): 컨트랙트를 배포하는 주체(보통 NFT의 소유자)의 주소를 seller 변수에 저장합니다. payable 키워드는 이 주소가 이더리움을 수신할 수 있도록 합니다.<br>
>> highestBid = _startingBid: 경매의 시작 입찰 금액을 highestBid 변수에 설정합니다. 이 금액은 경매 동안 다른 입찰자들이 이 금액 이상을 제시해야 하는 최소 금액을 정의합니다.<br><br>
```solidity
constructor(address _nft, uint256 _nftId, uint256 _startingBid) {
    nft = IERC721(_nft);
    nftId = _nftId;

    seller = payable(msg.sender);
    highestBid = _startingBid;
}
```

### 7. function 생성
> #### 1. start 함수
> - 접근 제한자:<br>
>> external 키워드는 이 함수가 오직 외부에서만 호출될 수 있음을 나타냅니다. 즉, 다른 컨트랙트나 트랜잭션을 통해서만 이 함수에 접근할 수 있습니다.<br>
조건 검사:<br>
> - require(!started, "started"): 이 조건은 started 변수가 false일 경우에만 함수가 실행되도록 합니다. 만약 이미 started가 true이면 "started"라는 에러 메시지와 함께 실행이 중단됩니다.<br>
>> require(msg.sender == seller, "not seller"): 이 조건은 함수를 호출하는 주체(msg.sender)가 seller 변수에 저장된 주소와 일치할 때만 함수가 실행되도록 합니다. 만약 주소가 일치하지 않으면 "not seller"라는 에러 메시지와 함께 실행이 중단됩니다.<br>
> - NFT 이전:<br>
>> nft.transferFrom(msg.sender, address(this), nftId): IERC721 인터페이스의 transferFrom 함수를 호출하여 msg.sender로부터 현재 컨트랙트(address(this))로 nftId에 해당하는 NFT를 이전합니다. 이는 경매 컨트랙트(English Auction Contract)가 NFT의 임시 소유권을 가지게 하여, 경매가 종료될 때 승리자에게 NFT를 안전하게 전송할 수 있도록 합니다.<br>
> - 경매 상태 업데이트:<br>
>> started = true: 경매가 시작되었음을 나타내기 위해 started 변수를 true로 설정합니다.<br>
>> endAt = block.timestamp + 7 days;: endAt 변수에 현재 블록 타임스탬프에 7일을 더한 값을 저장합니다(초 단위로 저장됩니다). 이는 경매 종료 시점을 설정합니다.<br>
> - 이벤트 발생:<br>
>> emit Start(): Start 이벤트를 발생시켜 경매의 시작을 외부에 알립니다. 이 이벤트는 경매 참여자들에게 경매가 시작되었음을 알리는 신호로 작용합니다.<br><br>
```solidity
function start() external {
    require(!started, "started");
    require(msg.sender == seller, "not seller");

    nft.transferFrom(msg.sender, address(this), nftId);
    started = true;
    endAt = block.timestamp + 7 days;

    emit Start();
}
```
> #### 2. bid 함수
> - 접근 제한자와 페이어블(payable):<br>
>> external 키워드는 이 함수가 오직 외부에서만 호출될 수 있음을 나타냅니다.<br>
>> payable 키워드는 이 함수가 이더리움을 전송받을 수 있도록 하며, 이는 입찰 시 전송되는 금액을 받기 위해 필요합니다.<br>
> - 조건 검사:<br>
>> require(started, "not started");: 경매가 시작되었는지 확인합니다. 만약 started가 false이면 "not started"라는 메시지와 함께 실행을 중단합니다.<br>
>> require(block.timestamp < endAt, "ended"): 현재 시간이 경매 종료 시간 endAt보다 이전인지 확인합니다. 만약 이미 경매가 종료되었다면 "ended"라는 메시지와 함께 실행을 중단합니다.<br>
>> require(msg.value > highestBid, "value < highest"): 전송된 금액(msg.value)이 현재의 최고 입찰 금액(highestBid)보다 큰지 확인합니다. 만약 그렇지 않다면 "value < highest"라는 메시지와 함께 실행을 중단합니다.<br>
> - 최고 입찰자 갱신:<br>
>> if (highestBidder != address(0)): 이전 최고 입찰자가 존재하는 경우 (즉, highestBidder 주소가 0 주소가 아닌 경우)<br>
>> bids[highestBidder] += highestBid: 이전 최고 입찰자의 입찰금을 bids 매핑에 더해 입찰금을 갱신합니다. 이를 통해 이전 최고 입찰자가 다음에 입찰금을 철회할 수 있도록 합니다.<br>
> - 새로운 최고 입찰자 설정:<br>
>> highestBidder = msg.sender: 현재 함수를 호출한 주소(msg.sender)를 새로운 최고 입찰자로 설정합니다.<br>
>> highestBid = msg.value: 전송된 금액(msg.value)을 새로운 최고 입찰 금액으로 설정합니다.<br>
> - 이벤트 발생:<br>
>> emit Bid(msg.sender, msg.value): Bid 이벤트를 발생시켜 새로운 입찰 정보를 외부에 알립니다. 이 이벤트는 입찰이 성공적으로 이루어졌음을 나타내며, 입찰자와 입찰 금액을 포함합니다.<br><br>
```solidity
function bid() external payable {
    require(started, "not started");
    require(block.timestamp < endAt, "ended");
    require(msg.value > highestBid, "value < highest");

    if (highestBidder != address(0)) {
        bids[highestBidder] += highestBid;
    }

    highestBidder = msg.sender;
    highestBid = msg.value;

    emit Bid(msg.sender, msg.value);
}
```
> #### 3. withdraw 함수
> - 접근 제한자:<br>
>> external 키워드는 이 함수가 오직 외부에서만 호출될 수 있음을 나타냅니다. 즉, 다른 컨트랙트나 트랜잭션을 통해서만 이 함수에 접근할 수 있습니다.<br>
> - 입찰금 철회 로직:<br>
>> uint256 bal = bids[msg.sender]: 함수를 호출한 주소(msg.sender)의 현재 입찰금을 bids 매핑에서 조회하여 bal 변수에 저장합니다.<br>
>> bids[msg.sender] = 0: 해당 주소의 입찰금 정보를 bids 매핑에서 0으로 초기화하여, 다시 입찰금을 철회하려는 시도가 없도록 합니다.<br>
> - 금액 전송:<br>
>> payable(msg.sender).transfer(bal): 초기화된 입찰금 bal을 함수를 호출한 주소(msg.sender)로 전송합니다. payable 키워드는 이더리움 주소가 이더를 수신할 수 있게 해주며, transfer 함수를 사용해 안전하게 이더를 해당 주소로 송금합니다.<br>
> - 이벤트 발생:<br>
>> emit Withdraw(msg.sender, bal): Withdraw 이벤트를 발생시켜 입찰금 철회 정보를 외부에 알립니다. 이 이벤트는 입찰금 철회가 성공적으로 이루어졌음을 나타내며, 입찰금을 철회한 주소와 철회한 금액을 포함합니다.<br><br>
```solidity
function withdraw() external {
    uint256 bal = bids[msg.sender];
    bids[msg.sender] = 0;
    payable(msg.sender).transfer(bal);

    emit Withdraw(msg.sender, bal);
}
```
> #### 4. end 함수
> - 접근 제한자:<br>
>> external 키워드는 이 함수가 오직 외부에서만 호출될 수 있음을 나타냅니다. 즉, 다른 컨트랙트나 트랜잭션을 통해서만 이 함수에 접근할 수 있습니다.<br>
> - 조건 검사:<br>
>> require(started, "not started"): 경매가 시작되었는지 확인합니다. 만약 started가 false이면 "not started"라는 메시지와 함께 실행을 중단합니다.<br>
>> require(block.timestamp >= endAt, "not ended"): 현재 시간이 경매 종료 시간 endAt 이후인지 확인합니다. 만약 아직 경매 시간이 남아 있다면 "not ended"라는 메시지와 함께 실행을 중단합니다.<br>
>> require(!ended, "ended"): 경매가 이미 종료되었는지 확인합니다. 만약 ended가 true이면 "ended"라는 메시지와 함께 실행을 중단합니다.<br>
> - 경매 상태 업데이트:<br>
>> ended = true: 경매의 종료 상태를 true로 설정하여, 경매가 종료되었음을 나타냅니다.<br>
> - NFT 및 금액 전송:<br>
>> if (highestBidder != address(0)) : 최고 입찰자가 존재하는 경우<br>
>> nft.safeTransferFrom(address(this), highestBidder, nftId): IERC721 인터페이스의 safeTransferFrom 함수를 사용하여 NFT를 현재 컨트랙트에서 최고 입찰자의 주소로 안전하게 전송합니다.<br>
>> seller.transfer(highestBid): 최고 입찰 금액을 판매자의 주소로 전송합니다.<br>
>> else : 최고 입찰자가 없는 경우<br>
>> nft.safeTransferFrom(address(this), seller, nftId): NFT를 다시 판매자에게 전송합니다. safeTransferFrom 함수는 to 주소에서 토큰을 받았는지 확인하는 과정을 포함합니다.<br>
> - 이벤트 발생:<br>
>> emit End(highestBidder, highestBid): End 이벤트를 발생시켜 경매의 종료를 외부에 알립니다. 이 이벤트는 최고 입찰자와 최고 입찰 금액을 포함하여, 경매의 결과를 공개합니다.<br><br>
```soldity
function end() external {
    require(started, "not started");
    require(block.timestamp >= endAt, "not ended");
    require(!ended, "ended");

    ended = true;
    if (highestBidder != address(0)) {
        nft.safeTransferFrom(address(this), highestBidder, nftId);
        seller.transfer(highestBid);
    } else {
        nft.safeTransferFrom(address(this), seller, nftId);
    }

    emit End(highestBidder, highestBid);
}
```
## 전체 코드
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

interface IERC721 {
    function safeTransferFrom(address from, address to, uint256 tokenId)
        external;
    function transferFrom(address, address, uint256) external;
}

contract EnglishAuction {
    event Start();
    event Bid(address indexed sender, uint256 amount);
    event Withdraw(address indexed bidder, uint256 amount);
    event End(address winner, uint256 amount);

    IERC721 public nft;
    uint256 public nftId;

    address payable public seller;
    uint256 public endAt;
    bool public started;
    bool public ended;

    address public highestBidder;
    uint256 public highestBid;
    mapping(address => uint256) public bids;

    constructor(address _nft, uint256 _nftId, uint256 _startingBid) {
        nft = IERC721(_nft);
        nftId = _nftId;

        seller = payable(msg.sender);
        highestBid = _startingBid;
    }

    function start() external {
        require(!started, "started");
        require(msg.sender == seller, "not seller");

        nft.transferFrom(msg.sender, address(this), nftId);
        started = true;
        endAt = block.timestamp + 7 days;

        emit Start();
    }

    function bid() external payable {
        require(started, "not started");
        require(block.timestamp < endAt, "ended");
        require(msg.value > highestBid, "value < highest");

        if (highestBidder != address(0)) {
            bids[highestBidder] += highestBid;
        }

        highestBidder = msg.sender;
        highestBid = msg.value;

        emit Bid(msg.sender, msg.value);
    }

    function withdraw() external {
        uint256 bal = bids[msg.sender];
        bids[msg.sender] = 0;
        payable(msg.sender).transfer(bal);

        emit Withdraw(msg.sender, bal);
    }

    function end() external {
        require(started, "not started");
        require(block.timestamp >= endAt, "not ended");
        require(!ended, "ended");

        ended = true;
        if (highestBidder != address(0)) {
            nft.safeTransferFrom(address(this), highestBidder, nftId);
            seller.transfer(highestBid);
        } else {
            nft.safeTransferFrom(address(this), seller, nftId);
        }

        emit End(highestBidder, highestBid);
    }
}
```
#### Open in [REMIX](https://remix.ethereum.org/?#code=Ly8gU1BEWC1MaWNlbnNlLUlkZW50aWZpZXI6IE1JVApwcmFnbWEgc29saWRpdHkgXjAuOC4yNDsKCmludGVyZmFjZSBJRVJDNzIxIHsKICAgIGZ1bmN0aW9uIHNhZmVUcmFuc2ZlckZyb20oYWRkcmVzcyBmcm9tLCBhZGRyZXNzIHRvLCB1aW50MjU2IHRva2VuSWQpCiAgICAgICAgZXh0ZXJuYWw7CiAgICBmdW5jdGlvbiB0cmFuc2ZlckZyb20oYWRkcmVzcywgYWRkcmVzcywgdWludDI1NikgZXh0ZXJuYWw7Cn0KCmNvbnRyYWN0IEVuZ2xpc2hBdWN0aW9uIHsKICAgIGV2ZW50IFN0YXJ0KCk7CiAgICBldmVudCBCaWQoYWRkcmVzcyBpbmRleGVkIHNlbmRlciwgdWludDI1NiBhbW91bnQpOwogICAgZXZlbnQgV2l0aGRyYXcoYWRkcmVzcyBpbmRleGVkIGJpZGRlciwgdWludDI1NiBhbW91bnQpOwogICAgZXZlbnQgRW5kKGFkZHJlc3Mgd2lubmVyLCB1aW50MjU2IGFtb3VudCk7CgogICAgSUVSQzcyMSBwdWJsaWMgbmZ0OwogICAgdWludDI1NiBwdWJsaWMgbmZ0SWQ7CgogICAgYWRkcmVzcyBwYXlhYmxlIHB1YmxpYyBzZWxsZXI7CiAgICB1aW50MjU2IHB1YmxpYyBlbmRBdDsKICAgIGJvb2wgcHVibGljIHN0YXJ0ZWQ7CiAgICBib29sIHB1YmxpYyBlbmRlZDsKCiAgICBhZGRyZXNzIHB1YmxpYyBoaWdoZXN0QmlkZGVyOwogICAgdWludDI1NiBwdWJsaWMgaGlnaGVzdEJpZDsKICAgIG1hcHBpbmcoYWRkcmVzcyA9PiB1aW50MjU2KSBwdWJsaWMgYmlkczsKCiAgICBjb25zdHJ1Y3RvcihhZGRyZXNzIF9uZnQsIHVpbnQyNTYgX25mdElkLCB1aW50MjU2IF9zdGFydGluZ0JpZCkgewogICAgICAgIG5mdCA9IElFUkM3MjEoX25mdCk7CiAgICAgICAgbmZ0SWQgPSBfbmZ0SWQ7CgogICAgICAgIHNlbGxlciA9IHBheWFibGUobXNnLnNlbmRlcik7CiAgICAgICAgaGlnaGVzdEJpZCA9IF9zdGFydGluZ0JpZDsKICAgIH0KCiAgICBmdW5jdGlvbiBzdGFydCgpIGV4dGVybmFsIHsKICAgICAgICByZXF1aXJlKCFzdGFydGVkLCAic3RhcnRlZCIpOwogICAgICAgIHJlcXVpcmUobXNnLnNlbmRlciA9PSBzZWxsZXIsICJub3Qgc2VsbGVyIik7CgogICAgICAgIG5mdC50cmFuc2ZlckZyb20obXNnLnNlbmRlciwgYWRkcmVzcyh0aGlzKSwgbmZ0SWQpOwogICAgICAgIHN0YXJ0ZWQgPSB0cnVlOwogICAgICAgIGVuZEF0ID0gYmxvY2sudGltZXN0YW1wICsgNyBkYXlzOwoKICAgICAgICBlbWl0IFN0YXJ0KCk7CiAgICB9CgogICAgZnVuY3Rpb24gYmlkKCkgZXh0ZXJuYWwgcGF5YWJsZSB7CiAgICAgICAgcmVxdWlyZShzdGFydGVkLCAibm90IHN0YXJ0ZWQiKTsKICAgICAgICByZXF1aXJlKGJsb2NrLnRpbWVzdGFtcCA8IGVuZEF0LCAiZW5kZWQiKTsKICAgICAgICByZXF1aXJlKG1zZy52YWx1ZSA+IGhpZ2hlc3RCaWQsICJ2YWx1ZSA8IGhpZ2hlc3QiKTsKCiAgICAgICAgaWYgKGhpZ2hlc3RCaWRkZXIgIT0gYWRkcmVzcygwKSkgewogICAgICAgICAgICBiaWRzW2hpZ2hlc3RCaWRkZXJdICs9IGhpZ2hlc3RCaWQ7CiAgICAgICAgfQoKICAgICAgICBoaWdoZXN0QmlkZGVyID0gbXNnLnNlbmRlcjsKICAgICAgICBoaWdoZXN0QmlkID0gbXNnLnZhbHVlOwoKICAgICAgICBlbWl0IEJpZChtc2cuc2VuZGVyLCBtc2cudmFsdWUpOwogICAgfQoKICAgIGZ1bmN0aW9uIHdpdGhkcmF3KCkgZXh0ZXJuYWwgewogICAgICAgIHVpbnQyNTYgYmFsID0gYmlkc1ttc2cuc2VuZGVyXTsKICAgICAgICBiaWRzW21zZy5zZW5kZXJdID0gMDsKICAgICAgICBwYXlhYmxlKG1zZy5zZW5kZXIpLnRyYW5zZmVyKGJhbCk7CgogICAgICAgIGVtaXQgV2l0aGRyYXcobXNnLnNlbmRlciwgYmFsKTsKICAgIH0KCiAgICBmdW5jdGlvbiBlbmQoKSBleHRlcm5hbCB7CiAgICAgICAgcmVxdWlyZShzdGFydGVkLCAibm90IHN0YXJ0ZWQiKTsKICAgICAgICByZXF1aXJlKGJsb2NrLnRpbWVzdGFtcCA+PSBlbmRBdCwgIm5vdCBlbmRlZCIpOwogICAgICAgIHJlcXVpcmUoIWVuZGVkLCAiZW5kZWQiKTsKCiAgICAgICAgZW5kZWQgPSB0cnVlOwogICAgICAgIGlmIChoaWdoZXN0QmlkZGVyICE9IGFkZHJlc3MoMCkpIHsKICAgICAgICAgICAgbmZ0LnNhZmVUcmFuc2ZlckZyb20oYWRkcmVzcyh0aGlzKSwgaGlnaGVzdEJpZGRlciwgbmZ0SWQpOwogICAgICAgICAgICBzZWxsZXIudHJhbnNmZXIoaGlnaGVzdEJpZCk7CiAgICAgICAgfSBlbHNlIHsKICAgICAgICAgICAgbmZ0LnNhZmVUcmFuc2ZlckZyb20oYWRkcmVzcyh0aGlzKSwgc2VsbGVyLCBuZnRJZCk7CiAgICAgICAgfQoKICAgICAgICBlbWl0IEVuZChoaWdoZXN0QmlkZGVyLCBoaWdoZXN0QmlkKTsKICAgIH0KfQo&lang=en&optimize=false&runs=200&evmVersion=null&version=soljson-v0.8.25+commit.b61c2a91.js)

## 실행

#### 실행을 위해서는 ERC-721 NFT가 필용합니다.
토큰이 없을 경우 아래 ERC-721 smart contract를 사용하세요
```solidity
// SPDX-License-Identifier: MIT
// Compatible with OpenZeppelin Contracts ^5.0.0
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract MyToken is ERC721, Ownable {
    uint256 private _nextTokenId;

    constructor()
        ERC721("MyToken", "MTK")
        Ownable(msg.sender)
    {}

    function safeMint(address to) public onlyOwner {
        uint256 tokenId = _nextTokenId++;
        _safeMint(to, tokenId);
    }
}
```
이 컨트랙트는 mint 할 때마다 토큰아이디가 1씩 증각합니다.

### 폴더 구조
아래와 같은 폴더 구조를 만들어야 합니다. <br><br>
<img src="https://github.com/Joon2000/English-Auction-Solidity/blob/main/images/%E1%84%91%E1%85%A9%E1%86%AF%E1%84%83%E1%85%A5%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9.png" width="250" height="450"></img>

### MyToken Mint
> MyToken contract를 컴파일 후 deploy합니다. 아래와 같이 세팅 후 deploy하면 됩니다.<br><br>
<img src="https://github.com/Joon2000/English-Auction-Solidity/blob/main/images/%E1%84%91%E1%85%A9%E1%86%AF%E1%84%83%E1%85%A5%E1%84%80%E1%85%AE%E1%84%8C%E1%85%A9.png" width="250" height="450"></img><br><br>
> NFT를 민트합니다. Deployed/Unpinned Contracts에서 deploy한 contract를 클릭합니다. 그 후 아래와 같은 버튼들 중 safeMint 버튼 옆에 주소 적는 빈칸에 본인이 원하는 주소(본인의 주소를 추천합니다)를 넣고 버튼을 클릭하면 됩니다.<br><br>
<img src="https://github.com/Joon2000/English-Auction-Solidity/blob/main/images/myTokenButton.png" width="250" height="450"></img><br><br>

### English Auction 실행
> 1. EnglishAuction contract를 deploy 합니다. 아래 이미지와 같이 만든 후 deploy 버튼을 누르면 됩니다. 이때 _NFT에는 위에서 deploy한 NFT 주소를, _NFTID에는 0(한개밖에 mint 안했기 때문에)을, _STARTINGBID에는 1ETH를 입력하기 위해서 1*10^18(wei 단위)을 넣어줍니다. <br><br>
<img src="https://github.com/Joon2000/English-Auction-Solidity/blob/main/images/EnglishAuctionDeploy.png" width="250" height="450"></img><br><br>
> 2. EnglishAuction contract가 IERC721의 transferFrom함수를 사용하기 위해서는 해당 nft의 ID에 대한 approve가 되어 있어야 합니다. MyToken contract의 approve 버튼 옆에 EnglishAuction contract의 주소를 입력 후 클릭합니다.<br><br>
<img src="https://github.com/Joon2000/English-Auction-Solidity/blob/main/images/MyTokenApprove.png" width="250" height="450"></img><br><br>
> #### Auction 버튼 조작
> <img src="https://github.com/Joon2000/English-Auction-Solidity/blob/main/images/EnglishAuctionButton.png" width="250" height="450"></img><br><br>
> 3. 먼저 auction을 시작하기 위히서 start버튼을 누릅니다. 이제 옥션이 시작한겁니다. 이때 사인하는 주소가 English Auction contract를 deploy한 주소와 동일해야 합니다. <br><br>
> 4. 파란색 버튼들을 실행해보면 모두 정상 작동하는거를 확인할 수 있습니다. 이때 아직 아무도 bid를 하지 않았기 때문에 이 주소는 비어 있습니다.<br><br>
<img src="https://github.com/Joon2000/English-Auction-Solidity/blob/main/images/EnglishAuctionBlueButton.png" width="250" height="450"></img><br><br>
> 5. 다른 주소로 바꿉니다. <br><br>
<img src="https://github.com/Joon2000/English-Auction-Solidity/blob/main/images/RemixAddress.png" width="250" height="450"></img><br><br>
> 6. Value를 2ETH로 바꾼 후 bid 버튼을 누릅니다. contract의 balance가 2ETH로 바뀌고 highestBid와 highestBidder가 바뀐 것을 확인할 수 있습니다.<br><br>
<img src="https://github.com/Joon2000/English-Auction-Solidity/blob/main/images/bidValue.png" width="250" height="450"></img><br><br>
> 7. 주소와 bid하는 value를 바꿔가면 옥션을 진행합니다.<br><br>
> 8. withdraw 함수는 해당 주소의 bid를 contract로부터 돌려받을 수 있습니다. 이때 highestBidder눈 withdraw 할 수 없습니다.<br><br>
> 10. 7일 이후에 end 함수를 누르면 옥션을 정지하고 highestbid만큼의 ETH가 seller의 주소로 전달됩니다. 
