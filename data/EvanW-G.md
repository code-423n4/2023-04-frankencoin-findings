[G-01] Avoid always reading ****`state variable`.**

In function `calculateCurrentFee`, it’s better to create a memory variable for `start` rather than repeatedly reading state variable

the state variable `minted` in `notifyChallengeSucceeded` also encounter the same issue.

```solidity
contracts/Position.sol
187: return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));

349: uint256 repayment = minted < volumeZCHF ? minted : volumeZCHF; // how much must be burned to make things even
```

Suggestion as an example:

```solidity
contracts/Position.sol
function calculateCurrentFee() public view returns (uint32) {
    uint256 exp = expiration;
    uint256 time = block.timestamp;
    uint256 _start = start;
    if (time >= exp){
        return 0;
    } else {
        return uint32(mintingFeePPM - mintingFeePPM * (time - _start) / (exp - _start));
    }
}
```

[G-02] params checking should be on the beginning of a function

function `checkCollateral` will revert the transaction if the collateral_ amount is not fulfilled. It’s better to put this line of code in the beginning like the LimitExceed checking

```solidity
contracts/Position.sol
198: checkCollateral(collateral_, price);
```

Suggestion:

```solidity
contracts/Position.sol
function mintInternal(address target, uint256 amount, uint256 collateral_) internal {
    if (minted + amount > limit) revert LimitExceeded();
    checkCollateral(collateral_, price);
    zchf.mint(target, amount, reserveContribution, calculateCurrentFee());
    minted += amount;

    emitUpdate();
}
```

[G-03] ****Avoid using `state variable` in emit (130 gas)**

Emit the memory variable or calldata variable instead.

```solidity
contracts/Position.sol

287: emit MintingUpdate(collateralBalance(), price, minted, limit);
```

[G-04] Don’t need to cast uint as uint192 and uint 64

Cast it as uint192 and uint 64 won’t save any gas fee because it’s not in a struct. Also, when using elements that are smaller than 32 bytes, your contracts gas usage may be higher.

```solidity
75: uint192 private totalVotesAtAnchor;  // Total number of votes at the anchor time, see comment on the um
76: uint64 private totalVotesAnchorTime; // 40 Bit for the block number, 24 Bit sub-block time resolution
```