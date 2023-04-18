## use << instead of 2**

https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/Equity.sol#L253

```
require(totalSupply() < 2**128, "total supply exceeded");
```


## use ether rather than 10**18
```
uint256 internal constant ONE_DEC18 = 10**18;

ONE_DEC18 used in Equity.sol and MathUtils.sol, Position.sol

