### Use != 0 instead of > 0 for unsigned integer comparison	

These are some of the instances in the contract where this is applicable: 
[1](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L84) and [2](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L84), `totalSupply() > 0` can be updated to `totalSupply() != 0`

Another instance where this is applicable is in the `allowanceInternal` function of the Frankencoin contract - [3](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L104)

