## [NC-01] Event is missing `indexed` fields.
_Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it’s not necessarily best to index the maximum allowed per event (threefields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed._

Note: Using the `indexed` keyword for value types such as `uint`, `bool`, and `address` saves gas costs. However, this is only the case for value types, whereas indexing `bytes` and `strings` are more expensive than their unindexed version.

There are 4 instances of this issue:

[contracts/Position.sol#L41-L42-L43](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L41)
```solidity
    event PositionOpened(address indexed owner, address original, address zchf, address collateral, uint256 price);
    event MintingUpdate(uint256 collateral, uint256 price, uint256 minted, uint256 limit);
    event PositionDenied(address indexed sender, string message);
```

[contracts/MintingHub.sol#L48-L49-L50-L51-L52](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L48)
```solidity
    event ChallengeStarted(address indexed challenger, address indexed position, uint256 size, uint256 number);
    event ChallengeAverted(address indexed position, uint256 number);
    event ChallengeSucceeded(address indexed position, uint256 bid, uint256 number);
    event NewBid(uint256 challengedId, uint256 bidAmount, address bidder);
    event PostPonedReturn(address collateral, address indexed beneficiary, uint256 amount);
```

[contracts/Equity.sol#L91](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L91)
```solidity
    event Trade(address who, int amount, uint totPrice, uint newprice);
```

[contracts/Frankencoin.sol#L53](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L53)
```solidity
    event MinterApplied(address indexed minter, uint256 applicationPeriod, uint256 applicationFee, string message);
    event MinterDenied(address indexed minter, string message);
```
