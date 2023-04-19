## [L-01] Attempt to transfer the previous bid back to its bidder even though the potential new bid might revert because is not bidding enough to overbid the current highest bid

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L203-L205
Checking if challenge.bid (current highest bid) is greater than 0 will always end up transferring back the bid to the current highest bidder, but at the point it is still unknown if the new bid is actually high enough to overbid the current highest bid, such a validation happens later in the code, in line:
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L216
If the above check turns out to be false, the entire transaction will be reverted, thus, the transfer of funds to the current highest bidder will be reverted too, thus, this indicates that the transfer of funds to the current highest bidder should be done only if the new bid has been validated to be able to overbid the current highest one, if it is not high enough there is no a real reason to first transfer the funds to the current highest bidder.

### Recommendation
- The recommendation would be to remove Lines 203-205 (https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L203-L205), and instead, attempt to transfer the challenge.bid to the challenge.bidder only after it has been validated that the new bid is higher than the current one

   - Move line 204 (https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L204)  below line 216 (https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L216)

---
## [L-02] Using an incorrect index when wiping out tables during the execution of the restructureCapTable()

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L313
Inside the `for` that is intended to iterate over the `addressToWipe[]` parameter with the end goal to burn the FPS tokens of those addresses is using an incorrect index when determining the `current` address to be wiped out.
- Instead of making use of the iterative index of the for loop, the `i` variable, is using the `0` index, which means that all the iterations of the for loop will be trying to burn shares only from the address at the position 0

### Recommendation
- The recommendation would be to update the value of the index for the correct value, which would be the iterative `i` variable defined in the for loop
`address current = addressesToWipe[i]`
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L313