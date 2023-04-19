## Low 1: Suggested minter can be denied while already being an accepted minter
A suggested minter can be denied as long as `block.timestamp <= minters[_minter]` accordring to the [denyMinter()](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L152-L157) method of the `Frankencoin` contract.  
However, a suggested minter can already act as an accepted minter as soon as `block.timestamp >= minters[_minter]` according to the [isMinter()](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L293-L295) method of the `Frankencoin` contract.  
Therefore, we have an overlap at `block.timestamp == minters[_minter]`. Although it's unlikely that this edge case ever occurs, it's still something that needs to be fixed.

## Low 2: No way to handle compromised collateral after cooldown
Following scenario: There is an open position with a perfectly valid but heavily centralized collateral token and the cooldown period has already expired, i.e. the owner of the position can mint ZCHF and cannot be denied anymore. However, the collateral token is secretly compromised by an attacker who also happens to be the owner of the position.  
The attacker having full control over the collateral token might stop transfers which makes it impossible to challenge the position due to the `transferFrom()` call in the [launchChallenge()](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L140-L148) method of the `MintingHub` contract.  
Moreover, the attacker might mint additional collateral to the position.  
Consequently, there is nothing that can stop the attacker (owner of the position) from minting ZCHF until the `limit` (specified at [construction](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L50-L70) of position) is reached or the position expires naturally (see `alive` modifier of [mint()](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L177-L179) method).  

In conclusion, the impact is absolutely critical and leads to severe loss of funds, but the attack path is highly hypothetical, therefore I decided to submit this as low severity. Anyways, it's technically possible to exploit this attack vector and I suggest to discuss how to handle broken / compromised collateral tokens after the cooldown period.