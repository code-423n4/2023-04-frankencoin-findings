# Summary
## Low Risk
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|L-01       | Average block per day is lower than 7200 |4|
|L-02       | Minter can skip fees  |4|


## Non critical
|ID     | Finding| Instances |
|:----: | :---           |   :----:         |
|N-01       | A year doesn't equal 52 weeks | 1 |
|N-02       | Revert should be used instead of require when conition is always false | 1 |
|N-03       | Missing 0 address check for sender | 1 |
|N-04       | Panic exception 0x11 can occur when burning token  | 1 |
|N-05       | Bitshifting makes code complicated and is prone to errors  | 1 |
|N-06       | Using empty super function  | 1 |
# Details
## Low Risk
## [L-01] Average block per day is lower than 7200
[`MIN_HOLDING_DURATION`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L59) is calculated by doing 90*7200. However the average blocks per day is closer to 7100. Which mean the min holding duration will be more than 90 days.
```solidity
    uint256 public constant MIN_HOLDING_DURATION = 90*7200 << BLOCK_TIME_RESOLUTION_BITS;
```
## [L-02] Minter can skip fees
There are two `mint()` functions in the Frankencoin contract. They both mint frankencoins, however one has fees and the other one has not. Since there is no disadvantage for the minter to use the mint function without fees he will always use it and skip the fees.

[Frankencoin.sol#L165-L174](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L165-L174)
```solidity
   function mint(address _target, uint256 _amount, uint32 _reservePPM, uint32 _feesPPM) override external minterOnly {
      uint256 usableMint = (_amount * (1000_000 - _feesPPM - _reservePPM)) / 1000_000; // rounding down is fine
      _mint(_target, usableMint);
      _mint(address(reserve), _amount - usableMint); // rest goes to equity as reserves or as fees
      minterReserveE6 += _amount * _reservePPM; // minter reserve must be kept accurately in order to ensure we can get back to exactly 0
   }

   function mint(address _target, uint256 _amount) override external minterOnly {
      _mint(_target, _amount);
   }
```
## Non critical
## [N-01] A year doesn't equal 52 weeks
A year is 52 weeks + 1 day, which means that the date that the contract needs to be updated will be one day earlier every year (or 2 in a leap year).
[StablecoinBridge.sol#L29](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L29)

## [N-02] Revert should be used instead of require when conition is always false
[StablecoinBridge.sol#L81](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L81)
```diff
-    require(false, "unsupported token");
+    revert("unsupported token");
```
## [N-03] Missing 0 address check for sender
[ERC20.sol#L151-L159](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L151-L159): the comment says sender cannot be the zero address. However there is only a check for the recipient and not for the sender. If this meant to be, the comment should be removed.
## [N-04] Panic exception 0x11 can occur when burning token
When [burning](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L200-L206) the token there is no require statement to check if the amount is greater than the account balance. Because of this underflow can happen and will result in the Panic exception 0x11
```solidity
    function _burn(address account, uint256 amount) internal virtual {
        _beforeTokenTransfer(account, address(0), amount);

        _totalSupply -= amount;
        _balances[account] -= amount;
        emit Transfer(account, address(0), amount);
    }
```
## [N-05] Bitshifting makes code complicated and is prone to errors
While it saves gas to put multiple values in one storage slot using bitshifting, it also makes the code really complicated.
[Equity.sol#L76](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L76)
## [N-06] Using empty super function 
The function [`_beforeTokenTransfer()`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L112-L122) in Equity.sol also uses the super function [`_beforeTokenTransfer()`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L240-L241). However this function is empty so it will do nothing. The call to the super function should be removed.
```solidity
    function _beforeTokenTransfer(address from, address to, uint256 amount) override internal {
        super._beforeTokenTransfer(from, to, amount);
        if (amount > 0){
            // No need to adjust the sender votes. When they send out 10% of their shares, they also lose 10% of
            // their votes so everything falls nicely into place.
            // Recipient votes should stay the same, but grow faster in the future, requiring an adjustment of the anchor.
            uint256 roundingLoss = adjustRecipientVoteAnchor(to, amount);
            // The total also must be adjusted and kept accurate by taking into account the rounding error.
            adjustTotalVotes(from, amount, roundingLoss);
        }
    }
```