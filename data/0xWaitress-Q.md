1. Trade event emitted with msg.sender, which is always zchf contract 

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L249

```solidity
        emit Trade(msg.sender, int(shares), amount, price());
```
Recommendation
change it to `from`
```solidity
        emit Trade(from, int(shares), amount, price());
```

--- \n