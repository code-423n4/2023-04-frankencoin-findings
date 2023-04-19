[G-1] Cache `totalSuply()` in frankencoin.sol's suggestMinter function
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84
```solidity
if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();

if (_applicationFee < MIN_FEE && totalSupply() > 0) revert FeeTooLow();// @audit gas for totalsuply
```
[G-2] In `isMinter` and `returnColatral` function cache minters[\_minter]
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L293
```solidity
function isMinter(address _minter) override public view returns (bool){

return minters[_minter] != 0 && block.timestamp >= minters[_minter];//@audit cache minter gas
}
```
[G-3]` challage.size` and `challenge.challenger` is used 3 times  which is state vairable.
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L287


