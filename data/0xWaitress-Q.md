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

2. The MIN_HOLDING_DURATION is set to 90 * 7200 (blocks), which is more than 90 days on the context of ethereum. Ethereum currently have around 7100 blocks only, the configuration would lead to an effective 91 days, instead of 90.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L54-L59

```solidity
     * The minimum holding duration in blocks. You are not allowed to redeem your pool shares if you held them
     * for less than the minimum holding duration at average. For example, if you have two pool shares on your
     * address, one acquired 5 days ago and one acquired 105 days ago, you cannot redeem them as the average
     * holding duration of your shares is only 55 days < 90 days.
     */
    uint256 public constant MIN_HOLDING_DURATION = 90*7200 << BLOCK_TIME_RESOLUTION_BITS; // Set to 5 for local testing
```

Recommendation:
set it to a more precise block number