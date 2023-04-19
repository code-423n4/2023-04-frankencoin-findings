Custom errors are available from solidity version **0.8.4**. Increase version.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L9
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L12

**Equity.sol#redeem, MintingHub.sol#bid, MintingHub.sol#end, MintingHub.sol#openPosition, Position.sol#repay** ignore return value by **zchf.transfer** and **zchf.transferFrom**. Marked as low/qa because zchf token functions revert if they fail.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L279
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L199-L229
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L252-L276
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L88-L113
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L227-L230

**calculateAssignedReserve(...)** in Frankencoin.sol performs two times a multiplication after a division. Consider ordering multiplication before division.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L204-L213

**withdraw** functions in Position.sol ignores return value by IERC20(collateral).transfer. It is unlikely that a transfer from the contract to the user could return false but ensure that the transfer return value is checked. In this worst case if the token used as collateral return false, the user do not withdraw the collateral but the functions still succeeds.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L211

**bid(...)** function in MintingHub.sol ignores return value by **challenge.position.collateral().transfer**. It is unlikely that a transfer from the contract to the user could return false but ensure that the transfer return value is checked. In this worst case if the token used as collateral return false, the bidder transfers zchf tokens and receives nothing.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L211

**adjust(...)** function in Position.sol ignores return value by challenge.position.collateral().transferFrom. Function will revert in the minting phase thanks to checkCollateral function but the transfer will pass.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L132-L152

**burn(...)**, **burnFrom(...)**, **burnWithReserve(...)**, **mint(...)** functions in Frankecoin.sol should emit an event.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L194-L197
