[G-1] Use `uint32` for timestamp
Gas savings deployment - 20000 per slot saved
Gas savings function call - 5000 per slot saved
Use `uint32` for timestamp to tight pack variables and save 1 slot.
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L43
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L26-#30
