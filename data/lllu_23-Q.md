# [G-01]Use hardcode address instead `address(this)` 
## Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded`address`. Foundry’s `script.sol` and solmate’s `LibRlp.sol` contracts can help achieve this.

### References: 
https://book.getfoundry.sh/reference/forge-std/compute-create-address
https://twitter.com/transmissions11/status/1518507047943245824

There are 7 instances of this issue:

`  115: require(zchf.isPosition(position) == address(this), "not our pos");`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L115-#L118

` 142: IERC20(position.collateral()).transferFrom(msg.sender, address(this), _collateralAmount);`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L142

`225: zchf.transferFrom(msg.sender, address(this), _bidAmountZCHF);`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L225

` 55: original = address(this);`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L55

`145: chf.transferFrom(msg.sender, address(this), amount);`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L45

`51: require(chf.balanceOf(address(this)) <= limit, "limit");`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L51

`79: burnInternal(address(this), from, amount);`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L79

## Recommended Mitigation Steps
Use hardcoded `address`.

# [G-02]`<x> += <y>` Costs More Gas Than ` <x> = <x> + <y>` For State Variables

## References: 
https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8

There are 4 instances of this issue:
` 291: pendingReturns[collateral][challenge.challenger] += challenge.size;`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L291

`196: minted += amount; `
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L196

`295: challengedAmount += size;`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L295

`199: _votes += votes(current);`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L199

# [G-03]`++i`/`i++` should be` unchecked{++i}`/`unchecked{i++}` when it is not possible for them to overflow, as is the case when used in `for`- and `while`-loops

## The `unchecked` keyword is new in solidity version `0.8.0`, so this only applies to that version or higher, which these instances are. This saves` 30-40 gas` per loop.


There are 2 instances of this issue:

`192: for (uint i=0; i<helpers.length; i++){
 193: address current = helpers[i];
 194: require(current != sender);
 195: require(canVoteFor(sender, current));
 196: for (uint j=i+1; j<helpers.length; j++){
 197: require(current != helpers[j]); // ensure helper unique
            }`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L192-#L198

`312: for (uint256 i = 0; i<addressesToWipe.length; i++){
 313: address current = addressesToWipe[0];
 314: _burn(current, balanceOf(current));
        }`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L312-#L315

# [G-04] Use double `if` statements instead of `&&`
## If the `if` statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

There are 2 instances of this issue:

` 84: if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort(); `
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84

` 85: if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L85


# [G-05] Optimize names to save gas
## Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because `22 gas` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

See more here.
https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92


## Proof Of Concept
All in-scope contracts.

Recommended Mitigation Steps
Find a lower method ID name for the most called functions for example `Call()` vs .`Call1()` is cheaper by 22 gas.

For example, the function IDs in the `Gauge.sol` contract will be the most used; A lower method ID may be given.

# [G-06] Use solidity version 0.8.19 to gain some gas boost
## Upgrade to the latest solidity version 0.8.19 to get additional gas savings.

See latest release for reference:
 https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/

## Proof Of Concept
` 2: pragma solidity ^0.8.0;`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L2

`9: pragma solidity ^0.8.0;`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L9

`2: pragma solidity ^0.8.0;`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L2

`2: pragma solidity ^0.8.0;`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L2

`2: pragma solidity ^0.8.0;`
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L2


