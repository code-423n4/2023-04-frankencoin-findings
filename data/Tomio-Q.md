Title: Integer over/underflow

Proof of Concept:
[Position.sol#L98](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L98)
the `reduceLimitForClone` function calculates a `reduction` value that could result in an underflow if the `_minimum` parameter is too high. Consider add safemath to avoid these issues.
________________________________________________________________________



