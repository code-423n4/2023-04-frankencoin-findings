L-1) Challengers can call end function of MintingHub.sol by setting postponeCollateralReturn to false. It overrides the purpose of postponing the return of collateral to the challenger.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L252

NC-1) Pro-rated fee is applied at the time of zchf minting, but system doesn't return remaining pro-rated fee when zchf tokens are burned. It's a loss to the user when he mints and returns them early, or does multiple mint/burn cycles.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L181-L189