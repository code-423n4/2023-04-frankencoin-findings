Title: `reduction` could cause an underflow

Proof of Concept:
[Position.sol#L98](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L98)

Recommended Mitigation Steps:
first check if `_minimum` is greater than or equal to `limit - minted` and revert if it is.
```   
    require(_minimum <= limit - minted, "..");
```
________________________________________________________________________

Title: Use single comparison operation instead of two is more gas efficient

Proof of Concept:
[Position.sol#L53](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L53)

Recommended Mitigation Steps:
Change to:
```
	require(initPeriod > 2 days); // >2 is the same as >=3.
```
________________________________________________________________________

Title: Redundant check

Proof of Concept:
[StablecoinBridge.sol#L50](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L50)
Consider remove it, since this expiration time is set in the constructor and cannot be changed, so it is already verified in the `horizon` variable.
________________________________________________________________________