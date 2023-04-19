## [N-01] Many functions do not check the `transfer()` return boolean.

Functions that do not check the `transfer()` return boolean risk executing even if the transfer is not successful. For example,

```
    function redeem(address target, uint256 shares) public returns (uint256) {
        require(canRedeem(msg.sender));
        uint256 proceeds = calculateProceeds(shares);
        _burn(msg.sender, shares);
        zchf.transfer(target, proceeds);
        emit Trade(msg.sender, -int(shares), proceeds, price());
        return proceeds;
    }
```

`redeem()` does not check to see if the transfer went through successfully. In addition, a burn occurs prior to the transfer. An attacker may find a way to burn his shares without getting his proceeds. This may sound like it would do more damage to him than the protocol, however, it can cause an imbalance in your calculations. Further, the Trade event will still be broadcast, potentially disrupting any web3 applications surrounding your project, as they anticipated that the proceeds had been transferred.

Here are all unchecked transfers:

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L279
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L204
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L211
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L263
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L284
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L294
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#69
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L87
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L225
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L254
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L283-L285

Recommended mitigation:
Add require statements to the transfers like such:

`require(zchf.transfer(target,proceeds), “Transfer failed”);`

## [N-02] Dust gets locked in the FPS contract.

2 shares must always remain on the network. That is to say, if Alice does the first deposits and gets 1000 shares & then tries to immediately transfer them back, she will only be able to transfer 998 shares.
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L290-L297

Recommended Mitigation:
Do not require there to be 2 shares left within the system.

## [N-03] `calculateAssignedReserve()` does a division prior to a multiplication.

To preserve precision, it is recommended that you multiply before dividing.

I am submitting this as non-critical because we are dealing with very small rounding errors.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L204-L213


Recommended Mitigation:

Move the division of 1000000 into the if else statement. This way, we can multiply by currentReserve before doing the two divisions.

```
function calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) internal view returns (uint256) {

    uint256 theoreticalReserve = _reservePPM* mintedAmount;
    uint256 currentReserve = balanceOf(address(reserve));
    if (currentReserve < minterReserve()){
        return theoreticalReserve*currentReserve/minterReserve()/1000000;
    } else {
        return theoreticalReserve/1000000;
    }
}
```

## [N-04] `_cubicRoot()` function does not check for max iterations.

Newton-Raphson technique is known to diverge for edge cases.

The Newton-Raphson method is an extremely robust method for root finding, and can be implemented relatively easily. Further, we have quadratic convergence to the root.

Still, it is known to potentially diverge on specific edge cases and not give the intended answers.

Source:
https://web.mit.edu/10.001/Web/Course_Notes/NLAE/node6.html
https://forum.openzeppelin.com/t/calculating-roots-in-solidity/1595/5

Recommendation:

Like in the openzeppelin source, add a max iteration so that in the case of divergence, the user does not get drained for max gas.