## [NC-1] Unused imports should be removed
Its recommended to remove unused imports for better readability of the code
and also saves some gas

There are 5 instances of it and their permalinks of them are below
[MintingHub.sol#L5](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L5) 
import "./IReserve.sol";

[MintingHub.sol#L7](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L7)
import "./Ownable.sol";

[Equity.sol#L7](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L7)
import "./IERC677Receiver.sol";

[StablecoinBridge.sol#L5](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L5)
import "./IERC677Receiver.sol";

[PositionFactory.sol#L5](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L5)
import "./IFrankencoin.sol";
