# Quality Assurance / Low Impact

## Use most recent compiler version or at least 0.8.4 in pragma statements

Custom errors are used which have been introduced in Solidity 0.8.4, but contracts allow for earlier compiler versions (^0.8.0).

### Proof of concept

```
pragma solidity ^0.8.0;
...
error NotOwner();
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L9
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L25

## Unused imports

[MintingHub imports Ownable](https://github.com/code-423n4/2023-04-frankencoin/blob/f86279e76fd9f810d2a25243012e1be4191a547e/contracts/MintingHub.sol#L7), but not using / inheriting from it. Can be deleted.

[PositionFactory imports IFrankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/f86279e76fd9f810d2a25243012e1be4191a547e/contracts/PositionFactory.sol#L5) but not using it. Can be deleted.

[Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L7) and [StablecoinBridge.sol import IERC677Receiver.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L6), but are not implementing it, instead redefining the `onTokenTransfer` function. Both contracts should inherit from `IERC677Receiver.sol` and override the `onTokenTransfer` function.

## require() / revert() statements should have descriptive reason strings [NC-22]

Two additional occurences in addition to Automated Findings [NC-22]:

https://github.com/code-423n4/2023-04-frankencoin/blob/f86279e76fd9f810d2a25243012e1be4191a547e/contracts/Equity.sol#L195

https://github.com/code-423n4/2023-04-frankencoin/blob/f86279e76fd9f810d2a25243012e1be4191a547e/contracts/Position.sol#L53

Consider adding custom errors instead of reason strings to save gas.

## Prevent division by 0 [LOW‑7][MED-2]

Additional occurence in addition to automated findings:  
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L209

## ERC20.sol does not add increaseAllowance / decreaseAllowance functions

The documentation states that 
```
Finally, the non-standard`decreaseAllowance`and`increaseAllowance` functions have been added to mitigate the well-known issues around setting allowances.
```
([source](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L36-L38)), but these functions are not present in the `ERC20` contract

### Impact

When an allowance is increased or decreased, the approved person could take advantage of both the old and the new allowance, as described [here](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit)

## Constructor in Ownable.sol was removed

... in comparison to the standard Open Zeppelin Ownable contract. This is not listed as one of the [modifications](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L5-L7) being made.

As a result, if `Ownable.sol` was deployed as a standalone contract, the owner would be `address(0)` because the default value of 0 is assigned to the owner variable, instead of `msg.sender` being set as owner in the constructor.

The documentation states ([source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/8d633cb7d169f2f8595b273660b00b69e845c2fe/contracts/access/Ownable.sol#L13-L14)):
```
By default, the owner account will be the one that deploys the contract. This can later be changed with {transferOwnership}.
``` 
, which is incorrect. By default, ownership is assigned to `address(0)`. The inheriting contract has the responsibility of setting the `owner` variable correctly in the constructor.

### Proof of concept

Concept can be tested by deploying the `Ownable.sol` contract separately, e.g. on Remix. The owner of the contract is `address(0)` and as a result, ownership can't be transferred.

### Recommendations

To avoid problems if `Ownable` is reused later, this modification should be clearly stated, along with a notice in the documentation that the contract that inherits from `Ownable` has the responsibility of setting the correct owner in the constructor through the `setOwner` function.  
In the scope of the project this is handled correctly. Only `Position.sol` inherits from `Ownable`, which sets the owner in the constructor / initializer.

Constructors can not be virtual, but consider marking the `Ownable` contract as abstract to further indicate that an implementation is required.

## Code repetition

```
uint256 powX3 = _mulD18(_mulD18(x, x), x);
``` 
could make use of the internal `_power3` function instead:  
```
uint256 powX3 = _power3(x);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MathUtil.sol#L24
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MathUtil.sol#L39-L41

## Change event parameter

To mint Frankencoin Pool Shares, a user has to call the `transferAndCall` function in `Frankencoin.sol`, which calls the `onTokenTransfer`function in `Equity.sol`.  
This function emits an event  
```
emit Trade(msg.sender, int(shares), amount, price())
```  
As the `msg.sender` is always the Frankencoin contract (permissioned function), there is no added value in logging `msg.sender`. Should be replaced with `from`, the address buying the pool shares.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L249

## Named mappings for better readability

Using named mappings introduced in [Solidity 0.8.18](https://docs.soliditylang.org/en/v0.8.18/types.html#mapping-types) sets the name field for the inputs and outputs in the ABI for the mapping’s getter, making it easier for the user to understand what the expected input value and the output value is. For example a user reading from the contract on Etherscan will be presented with the input fields `address collateral` and `address beneficiary`instead of two `address` fields.

```
mapping(address collateral => mapping(address beneficiary => uint256 amount)) public pendingReturns;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L37

No impact on deployment cost

## Incorrect description of storage variable

```
IPositionFactory private immutable POSITION_FACTORY; // position contract to clone
```

https://github.com/code-423n4/2023-04-frankencoin/blob/f86279e76fd9f810d2a25243012e1be4191a547e/contracts/MintingHub.sol#L28
