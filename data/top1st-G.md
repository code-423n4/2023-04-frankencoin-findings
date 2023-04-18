## use << instead of 2**

https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/Equity.sol#L253

```
require(totalSupply() < 2**128, "total supply exceeded");
```