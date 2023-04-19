# Gas Optimizations

## Summary

|       Number    | Issue        | Instances     |
| :-------------: |:-------------|:-------------:|
| 1 | Use `memory` pointers instead of `storage` | 3 |
| 2 | `storage` variable should be cached into `memory` variables instead of re-reading them  | 19 |
| 3 | Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct | 2 |
| 4 | Use `unchecked` blocks to save gas  | 5 |
| 5 | Input check statements should be placed at the start of the functions | 1 |

## Findings

### 1- Use `memory` pointers instead of `storage` :

Copying structs from `storage` to `memory` is cheaper as soon as there are more reads on the `storage` pointer as there are properties in the struct.

There are 3 instances of this issue :

**File: MintingHub.sol**

[Line 157](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L157)
```
Challenge storage challenge = challenges[_challengeNumber];
```

[Line 200](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L200)
```
Challenge storage challenge = challenges[_challengeNumber];
```

[Line 253](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L253)
```
Challenge storage challenge = challenges[_challengeNumber];
```


### 2- `storage` variable should be cached into `memory` variables instead of re-reading them :

The instances below point to the second+ access of a state variable within a function, the caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read, thus saves **100gas** for each instance.

There are 19 instances of this issue :

**File: ERC20.sol** [Line 155](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L155)

In the code linked above the value of `_balances[sender]` is read multiple times (2) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: Frankencoin.sol** 

[Line 84-85](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84-L85)

In the code linked above the value of `totalSupply()` is read multiple times (2) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 207-209](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L207-L209)

In the code linked above the value of `minterReserve()` is read multiple times (2) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 294](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L294)

In the code linked above the value of `minters[_minter]` is read multiple times (2) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: MintingHub.sol** 

[Line 158-176](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L158-L176)

In the code linked above the value of `challenge.challenger` is read multiple times (3) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 161-176](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L161-L176)

In the code linked above the value of `challenge.position` is read multiple times (3) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 171-176](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L171-L176)

In the code linked above the value of `challenge.size` is read multiple times (2) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 201-218](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L201-L218)

In the code linked above the value of `challenge.end` is read multiple times (2) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 202-211](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L202-L211)

In the code linked above the value of `challenge.size` is read multiple times (2) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 203-204](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L203-L204)

In the code linked above the value of `challenge.bid` is read multiple times (2) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 254-272](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L254-L272)

In the code linked above the value of `challenge.challenger` is read multiple times (2) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 259-263](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L259-L263)

In the code linked above the value of `challenge.bidder` is read multiple times (3) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 260-274](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L260-L274)

In the code linked above the value of `challenge.bid` is read multiple times (3) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 291-294](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L291-L294)

In the code linked above the value of `challenge.challenger` is read multiple times (3) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 291-294](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L291-L294)

In the code linked above the value of `challenge.size` is read multiple times (3) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

**File: Position.sol**

[Line 141-142](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L141-L142)

In the code linked above the value of `minted` is read multiple times (2) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 149-150](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L149-L150)

In the code linked above the value of `minted` is read multiple times (2) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 241](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L241)

In the code linked above the value of `minted` is read multiple times (2) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 349](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L349)

In the code linked above the value of `minted` is read multiple times (2) from storage and its value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.


### 3- Multiple address/IDs mappings can be combined into a single mapping of an address/id to a struct :

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

There are 2 instances of this issue :

**File: Equity.sol** [Line 83-88](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L83-L88)
```solidity
83:     mapping (address => address) public delegates;
88:     mapping (address => uint64) private voteAnchor; 
```


### 4- Use `unchecked` blocks to save gas :

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

There are 5 instances of this issue:

**File: ERC20.sol** [Line 273](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L156)
```solidity
_balances[sender] -= amount;
```

In the code linked above the operation cannot underflow due to the checks that preceeds it [Line 155](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L155) so it should be marked as `unchecked`.


**File: Equity.sol** [Line 294](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L294)
```solidity
uint256 newTotalShares = totalShares - shares;
```

In the code linked above the operation cannot underflow due to the checks that preceeds it [Line 293](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L293) so it should be marked as `unchecked`.


**File: Frankencoin.sol** [Line 144](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L144)
```solidity
return balance - minReserve;
```

In the code linked above the operation cannot underflow due to the checks that preceeds it [Line 141](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L141) so it should be marked as `unchecked`.


**File: Position.sol** 

[Line 242](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L242)
```solidity
minted -= amount;
```

In the code linked above the operation cannot underflow due to the checks that preceeds it [Line 241](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L241) so it should be marked as `unchecked`.


[Line 196](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L196)
```solidity
minted += amount;
```

In the code linked above the operation cannot overflow due to the checks that preceeds it [Line 194](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L194) so it should be marked as `unchecked`.


### 5- Input check statements should be placed at the start of the functions :

The check statements on the functions input values should be placed at the beginning of the functions to avoid stack too deep errors and save gas.

There is 1 instance of this issue:

**File: MintingHub.sol** [Line 109](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L109)
```solidity
require(_initialCollateral >= _minCollateral, "must start with min col");
```