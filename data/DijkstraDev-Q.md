#### [QA-01] Repositioning the `require(...)` statement.
As a general rule, checks that can fail should be placed as high as possible. In this case, a new `Position` is created first, followed by registering and transferring assets from the user, and only then checking if `_initialCollateral >= _minCollateral`, which would make more sense to check _before_ creating the new position. Additionally, unnecessary gas expenses for the function caller are reduced in case of failure.
[MintingHub.sol#L109](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/MintingHub.sol#L109)

#### [QA-02] Unnecessary casting.
Casting to `Position` is not necessary because the `clonePosition()` and `createClone()` functions match the data type to be returned.
[PositionFactory.sol#L32](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/PositionFactory.sol#L32)
[PositionFactory.sol#L37](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/PositionFactory.sol#L37)

Casting to `IPosition` is not necessary, as it is never used as an instance but only its _address_.
[MintingHub.sol#L92](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/MintingHub.sol#L92)

Casting to `IReserve` is not necessary, as `Position.reserve()` already returns an instance of `IReserve`.
[Position.sol#L111](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/Position.sol#L111)
[Frankencoin.sol#L31](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/Frankencoin.sol#L31)

#### [QA-03] Lack of feedback on potential errors.
There are many `require(...)` statements that do not contain a _message_ indicating to the user where the problem might be. This is not only useful for users who are interacting through a web page, but also for those who are automating liquidation processes and others.

[Find all occurrences]

#### [L-01] Lack of validation of values used to generate new positions.
Currently, only `_minCollateral` and `initPeriod` are being validated.
[MintingHub.sol#L88](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/MintingHub.sol#L88)
[PositionFactory.sol#L13](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/PositionFactory.sol#L13)
[Position.sol#L50](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/Position.sol#L50)