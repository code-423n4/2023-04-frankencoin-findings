### [G-01] Use != 0 instead of > 0 for unsigned integer comparison

When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0.  

[contracts/Equity.sol#L114](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L114)  
```
if (amount > 0){
```
[contracts/Frankencoin.sol#L84](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84)  
[contracts/Frankencoin.sol#L85](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L85)  
[contracts/Frankencoin.sol#L104](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L104)  
[contracts/MintingHub.sol#L203](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L203)  
[contracts/Position.sol#L389](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L389)  

### [G-02] Reverting should be done as soon as possible

[contracts/MintingHub.sol#L171](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L171)  
```
require(challenge.size >= min);
```

### [G-03] Splitting require() statements that use && saves gas

Instead of a single require statement with multiple conditions, multiple require statements with one condition per statement should be used to save gas.  

[contracts/ERC20PermitLight.sol#L56](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L56)  
```
require(recoveredAddress != address(0) && recoveredAddress == owner, "INVALID_SIGNER");
```

### [G-04] Cache array length outside of loop

Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.  

[contracts/Equity.sol#L192](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L192)  
```
for (uint i=0; i<helpers.length; i++){
```
[contracts/Equity.sol#L196](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L196)  
[contracts/Equity.sol#L312](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L312)  
[contracts/MintingHub.sol#L143](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L143)  
[contracts/MintingHub.sol#L174](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L174)  
