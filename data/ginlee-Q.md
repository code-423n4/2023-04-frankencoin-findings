[L1] recipient can't be sender
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L151-L159

not only recipient can't be zero address, it can not be sender also, or else function is useless

Mitigation
require(recipient != sender, "recipient can't be sender")