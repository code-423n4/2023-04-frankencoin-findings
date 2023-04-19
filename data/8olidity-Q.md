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
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L360-L362
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

## Vulnerable to cross-chain replay attacks due to static DOMAIN_SEPARATOR
Please consult EIP1344 for more details: https://eips.ethereum.org/EIPS/eip-1344#rationale

ref: https://github.com/code-423n4/2021-04-maple-findings/issues/2


https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L61-L71

```solidity
    function DOMAIN_SEPARATOR() public view returns (bytes32) {
        return
            keccak256(
                abi.encode(
                    //keccak256("EIP712Domain(uint256 chainId,address verifyingContract)");
                    bytes32(0x47e79534a245952e8b16893a336b85a3d9ea9fa8c573f3d803afb92a79469218),
                    block.chainid,
                    address(this)
                )
            );
    }
```


## When the equity is 0, calculateSharesInternal() will have an error of dividing by 0

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L268

When `balanceOf(address(reserve)) < minterReserve()`, the equity function will return 0.

```solidity
function equity() public view returns (uint256) {
  uint256 balance = balanceOf(address(reserve));
  uint256 minReserve = minterReserve();
  if (balance <= minReserve){
    return 0;
  } else {
    return balance - minReserve;
  }
```

Then in the `calculateSharesInternal` calculation, if the value of `zchf.equity()` is 0, then there will be an error of dividing by 0.

```solidity
* @notice Calculate shares received when depositing ZCHF
 * @param investment ZCHF invested
 * @return amount of shares received for the ZCHF invested
 */
function calculateShares(uint256 investment) public view returns (uint256) {
    return calculateSharesInternal(zchf.equity(), investment);
}

function calculateSharesInternal(uint256 capitalBefore, uint256 investment) internal view returns (uint256) {
    uint256 totalShares = totalSupply();
    uint256 newTotalShares = totalShares < 1000 * ONE_DEC18 ? 1000 * ONE_DEC18 : _mulD18(totalShares, _cubicRoot(_divD18(capitalBefore + investment, capitalBefore)));//@audit
    return newTotalShares - totalShares;
}
```