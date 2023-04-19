[L-1] Unsafe transferFrom() on Arbitrary ERC20 Token

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L45

While this contract is only for stablecoins that are trusted, it may be worthwhile to implement a transfer similar to OpenZeppelin's [safeTransferFrom() > _callOptionalReturn()](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/8d633cb7d169f2f8595b273660b00b69e845c2fe/contracts/token/ERC20/utils/SafeERC20.sol#L122-L123) which takes into account tokens that don't revert on failure (like USDT).

A bad taken (such as one like USDT) would result in the transferFrom() failing without Revert; the [mintInternal()](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L46) will succeed. This would allow someone to mint() infinite `zchf` tokens which can be [burned](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L55) for `chf` tokens that they never deposited, effectively draining the contract of all `chf` tokens.

While unlikely and your documents/comments suggest you will only implement trusted coins that properly implement the ERC20 spec, it's always a good idea to implement safeguards in code rather than open yourself to human error in the future.