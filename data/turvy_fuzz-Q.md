### deny period should be capped to prevent user from filling the position with long deny periods
`public` function openPosition `1` calls openPosition `2` with setting the _initPeriodSeconds( the deny period duration) to a week(7 days)
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L64

but the issue is openPosition `2` is also `public` and can also be called by anyone which they can set the _initPeriodSeconds to any amount
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L91

Recommedation:
denyPeriod should be capped or set openPosition `2` to be an `internal` function instead