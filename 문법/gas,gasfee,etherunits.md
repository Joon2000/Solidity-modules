# Gas, Gas fee and Ethereum Units

### Gas
Gas는 Transaction을 실행하기 위해 얼마나 Ether가 필요한지를 나타내는 것입니다.
Ether = Gas spent * Gas price
- Gas는 계산 단위입니다.
- Gas spent는 Transcation 중에 사용된 가스 총량입니다.
- Gas price는 각 Gas마다 몇 Ether를 지불해야 되는지 나타내는 값입니다.

#### Gas price가 높은 Transaction은 블록 안에서 높은 우선순위를 갖습니다.
#### 사용되지 않은 Gas는 돌려받습니다.

### Gas Limit 
- gas limit : transaction에 최대로 사용할 가스를 정할 수 있습니다. 사용자가 정하는 값입니다.
- block gas limit : 하나의 블록에서 허용된 최대 가스 양입니다. 네트워크가 정하는 값입니다.

### EtherUnits
Ethereum에서 Ether Units는 주로 거래 금액을 나타내는 데 사용됩니다. Ether는 다양한 단위로 나눌 수 있으며, 서로 다른 규모의 금액을 표현합니다. Gas를 계산할 때 다양한 단위가 사용됩니다. Solidity 내에서 단위를 직관적으로 사용할 수 있습니다.

- Wei : Ethereum에서 가장 작은 단위로, 1 Ether = 10^18 Wei입니다.
- GWei : 1 Gwei = 10**9 Wei
- Ether : 기본 단위로, 1 Ether = 10**18 Wei입니다.

## 예제 코드
```bash
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract Gas {
    uint256 public i = 0;

    // 보낸 모든 가스를 소모하면 Transaction이 실패합니다. 
    // State 변화는 다시 원상태로 돌아옵니다. 
    // 사용된 Gas는 돌려받지 않습니다.
    function forever() public {
        // 모든 가스를 소모할 때까지 반복문을 실행하며 Transaction이 실패하도록 합니다.
        while (true) {
            i += 1;
        }
    }
}

```

#### bool : return 값이 True / False 입니다.
```bash
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract EtherUnits {
    uint256 public oneWei = 1 wei;
    uint256 public oneGwei = 1 gwei;
    uint256 public oneEther = 1 ether;

    // 1 wei == 1 확인하는 bool
    bool public isOneWei = (oneWei == 1);

    // 1 gwei == 1e9 (10**9) 확인하는 bool
    bool public isOneGwei = (oneGwei == 1e9);

    // 1 gwei == 1e9 (10**18) 확인하는 bool
    bool public isOneEther = (oneEther == 1e18);
}
```

## Remix에서 실습 
1. Remix에서 새로운 solidity 파일 생성해서 예제 코드를 복사 붙여넣기 합니다.
2. 예제 코드를 compile 후 deploy합니다.
3. 아래 버튼들이 제대로 동작하는지 확인합니다.

- Gas를 모두 다 소모해서 Transaction이 실패한 결과입니다.<br>
<img src= "https://github.com/Joon2000/Solidity-modules/blob/ade08c61928fa828413fb7b2a82fbfc1daa2af8f/images/ethereumunits/Gas.png" width="1000px" height="200px" 
  title="gas" alt="gas"><br/>

- Ethereum 단위들을 확인할 수 있습니다. <br>
<img src= "https://github.com/Joon2000/Solidity-modules/blob/ade08c61928fa828413fb7b2a82fbfc1daa2af8f/images/ethereumunits/EthereumUnits.png" width="250px" height="400px" 
  title="ethereumunits" alt="ethereumunits"><br/>
