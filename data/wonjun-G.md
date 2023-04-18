[GAS-1] Amounts should be checked for 0 before calling a transfer
Checking non-zero transfer values can avoid an expensive external call and save gas.

I suggest adding a non-zero-value check here:

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L282
