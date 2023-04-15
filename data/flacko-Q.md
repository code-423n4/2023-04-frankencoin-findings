https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L290-L297

## Impact

The `calculateProceeds` method guards against redeeming too many shares, ensuring there's always 1 share left for the total supply. Or at least that's what the comment on that line says.

`require(shares + ONE_DEC18 < totalShares, "too many shares"); // make sure there is always at least one share`

In reality it ensures there are at least 2 shares left as a total supply.

## Proof of Concept

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L290-L297

```solidity
function calculateProceeds(uint256 shares) public view returns (uint256) {
    uint256 totalShares = totalSupply();
    uint256 capital = zchf.equity();
    require(shares + ONE_DEC18 < totalShares, "too many shares"); // make sure there is always at least one share
    uint256 newTotalShares = totalShares - shares;
    uint256 newCapital = _mulD18(capital, _power3(_divD18(newTotalShares, totalShares)));
    return capital - newCapital;
}
```

Let's imagine the following scenario:

1. Alice is the only shareholder with a balance of 2 shares. Hence, total supply is 3 shares
2. She wants to redeem her 2 shares and passes `2` as argument for `shares`
3. The require expression will evaluate to `2 + 1 < 3` preventing Alice from redeeming
4. To actually redeem, she has to redeem 1 shares, so that the require expression evaluates to true which means she has to leave 1 share on the table.

In a scenario where shares price is going big, knowing that you have to leave money in is unsettling and not entirely fair.

## Tools Used

Manual review

## Recommended Mitigation Steps

In the comparison in question, check if `shares + ONE_DEC18 <= totalShares` instead of `shares + ONE_DEC18 < totalShares`.