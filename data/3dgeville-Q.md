https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L31

The challenges array is used to store information about open challenges. However, there is no way to remove a challenge from the array once it has been added. This could lead to excessive storage usage if many challenges are initiated but never resolved. You may consider adding a function to allow the creator of a challenge to cancel it and remove it from the array.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L107

The INFINITY keyword is not defined. It seems like it should be replaced with type(uint256).max.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L84

In the suggestMinter function, the check totalSupply() > 0 is incorrect. It should check if totalSupply() > 1, because the initial minting of the token will make the total supply equal to 1, not 0. Alternatively, the check can be removed altogether, as it is not clear why it should be necessary.

The contract's constructor initializes the ERC20 token with 18 decimal places (i.e., ERC20(18)). When a new ERC20 token is created, an initial supply of 0 is assigned to the creator's address.
However, the Frankencoin contract does not have a designated address to receive the initial supply. Thus, the initial minting of the token, which occurs when the first user acquires some amount of the token, will set the total supply to 1.
After that, the total supply will increase or decrease as the token is minted or burned, respectively.


https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L117-L119

In the Frankencoin constructor, the ERC20 constructor is called with an argument of 18, which is the number of decimal places for the token. However, this is inconsistent with the use of minterReserveE6, which stores the minter reserve with 6 additional digits of accuracy. This could lead to rounding errors if the number of decimal places is changed in the future. Either the ERC20 constructor should be called with a different number of decimal places (e.g. 24), or the minterReserveE6 variable should be changed to store the reserve with the same number of decimal places as the token.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L45-L47

Instead of checking if owner != sender using an if statement, it is better to use a require statement, as follows:
function requireOwner(address sender) internal view {
    require(owner == sender, "Not owner");
}
This will provide a more informative error message to the caller of the function, in case the check fails.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L39

The VALUATION_FACTOR constant is set to 3. This determines the market cap of the reserve pool shares relative to the equity reserves. However, the code assumes that there will never be more than 10 million FPS. If this assumption is violated, it may lead to unexpected results. This could be an edge case that should have some mitigations.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L46

The QUORUM constant is set to 300 basis points, which is equivalent to 3%. This means that anyone who holds at least 3% of the holding-period-weighted reserve pool shares gains veto power and can veto new proposals. However, it's unclear how proposals are submitted and processed, so it's hard to say whether this mechanism is robust and effective. With a low market cap this means that there could be a dead lock I proposals meaning that the government mechanism fails. This should be valuated by the market cap which is change

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L59

The MIN_HOLDING_DURATION constant is set to 90 times 7200 left-shifted by BLOCK_TIME_RESOLUTION_BITS, which is 24. This means that the minimum holding duration is 90*7200/(2^24) blocks, which is roughly 1.2 days. This assumes that the average block time is around 15 seconds. If the block time changes significantly, this may affect the minimum holding duration and the redemption mechanism.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L75-L76

The totalVotesAtAnchor and totalVotesAnchorTime variables store the total number of votes at the anchor time and when the anchor time was. The code assumes that the block number can always be stored in 40 bits and the total supply cannot exceed 68 bits. However, this may become problematic if the total supply grows significantly, as it may cause overflow or other arithmetic errors.