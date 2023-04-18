## Suggestion to modify `>=` to `>`

In the `bid()` function, if the remaining time is less than 30 minutes, increase the time to allow other bidders more opportunities to bid.

```
uint256 earliestEnd = block.timestamp + 30 minutes;
if (earliestEnd >= challenge.end) {//@audit
    // bump remaining time like ebay does when last minute bids come in
    // An attacker trying to postpone the challenge forever must increase the bid by 0.5%
    // every 30 minutes, or double it every three days, making the attack hard to sustain
    // for a prolonged period of time.
    challenge.end = earliestEnd;
}

```

However, the `>=` operator can be replaced with `>`, because when `block.timestamp + 30 minutes == challenge.end`, there is no need to update the value of `challenge.end` again, since `earliestEnd` is already equal to `challenge.end`.



## The amount in the contract should not be used to determine whether the position is closed

Ideally a position should only be considered "closed" after its collateral has been withdrawn, then using isclosed is correct.
```solidity

    function collateralBalance() internal view returns (uint256){
        return IERC20(collateral).balanceOf(address(this));
    }


/**
     * A position should only be considered 'closed', once its collateral has been withdrawn.
     * This is also a good creterion when deciding whether it should be shown in a frontend.
   */
function isClosed() public view returns (bool) {
    return collateralBalance() < minimumCollateral;//@audit  
}
```


However, the attacker can send a certain amount of collateral assets to the contract so that the `collateralBalance>=minimumCollateral`, then it is impossible to use `isClosed` to determine whether a position has been closed.