## Low Issues

### L-1 Position_#251_`onlyOwner` modifier double check
The [`withdraw` function](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L1020-L1026) is an universal function to withdraw collateral and sweeping of other ERC20-like tokens. There is a double check `onlyOwner` in case of collateral because of the same modifier in the [`withdrawCollateral` function](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L263). I suggest just reverting calls with collateral addresses in the arguments and using this function only for sweeping.

### L-2 Equity_#293_Wrong comparison
There is the requirement to leave at least one share on the balance in the [line #293](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L293). It is correct using `<=` comparison instead of `<`.

### L-3 Equity_#313_Iterator is not used
The [loop](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L312-L315) in the `restructureCapTable` function doesn't use iterator in the line #313 in the `addressesToWipe` array. It is just a typo and it can't be the cause of any critical problems. But function work is not correct. I suggest changin `0` on the `i` iterator. 

### L-4 PositionFactory_#44_Answer from the `create` isn't handle 
The [`createClone` function](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L37-L46) contains the `create` call in the line #44, which can return `0` in case of error. The transaction flow will be reverted only farther in the [`transferFrom`](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L129) and only if the ERC20-like token does not support transfering on the `address(0x0)`. I suggest handling the return value.