## `StablecoinBridge` should give back the allowance when reaches to the `limit`
Imagine the below scenario:
1 . Alice approves `x` amount of XCHF tokens for the bridge and mints `x` amount of ZCHF
2 . Now bridge balance is reached to the `limit` amount (https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L51)
3 . Bob approves `y` tokens for the bridge
4 . Bob wants to mints `y` amount of ZCHF tokens and the transaction will be failed
5 . Bridge should set allowance value (from Bob) to 0 (not implemented)

Recommendation:
Implement step 5 and set allowance value to 0 when bridge limit is reached