Equity votes(sender, helpers) function inefficient uniqueness check 
(https://github.com/code-423n4/2023-04-Frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L196-L198)
Instead, create alreadyHelped mapping (address -> bool) setting it to true whenever a helpers vote are added to the total votes and reverting if it is true already.


suggestMinter function, do not check if totalSupply() > 0 twice, but only once before performing the FeeTooLow and periodTooShort checks
(https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L84-L85)
