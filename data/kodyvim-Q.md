# QA report:
## `suggestMinter` can be initially frontrunned.
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L83
`suggestMinter` does the following check.
```
if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
if (minters[_minter] != 0) revert AlreadyRegistered(); 
```
This means during deployment totalSupply() would be zero, an attacker or bot monitoring the mempool for such transaction could frontrun the call to suggestMinter setting their own minter address with _applicationPeriod = 0 and _applicationFee = 0, this would bypass the check since `&&` is used.
Recommendation:
Add a check `if (totalSupply() == 0 && msg.sender != INITIAL_DEPLOYER) revert NOT_AUTHORIZED;` to the `suggestMinter` function, with this you can also remove the other checks for `totalSupply()` saving gas as well.

## `expectedSize` parameter does not guard against frontrunners.
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L199
This parameter does not guard against frontrunners since MEV bot could read the calldata or follow the evm traces.
## Recommendations:
Follow a commit-Reveal Scheme.