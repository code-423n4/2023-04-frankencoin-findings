### [Gas-01] Function call Should Be Cached Rather Than Re-Calling It.
Below ```totalSupply()``` should be cached in ```memory```
```solidity
   function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {
+     uint256 _totalSupply = totalSupply();
-     if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort(); 
-     if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
+     if (_applicationPeriod < MIN_APPLICATION_PERIOD && _totalSupply > 0) revert PeriodTooShort(); 
+     if (_applicationFee < MIN_FEE  && _totalSupply > 0) revert FeeTooLow();
      if (minters[_minter] != 0) revert AlreadyRegistered();
      _transfer(msg.sender, address(reserve), _applicationFee);
      minters[_minter] = block.timestamp + _applicationPeriod;
      emit MinterApplied(_minter, _applicationPeriod, _applicationFee, _message); 
   }
```
*Instances(2)*
```
File:: contracts/Frankencoin.sol
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L85
```

### [Gas-02] Via Re-Ordering ```modifiers``` We Can Save Gas
```solidity
    function reduceLimitForClone(uint256 _minimum) external noChallenge noCooldown alive onlyHub returns (uint256) { // @audit short circuiting by reordering modifiers
        uint256 reduction = (limit - minted - _minimum)/2; // this will fail with an underflow if minimum is too high // @audit-info
        limit -= reduction + _minimum;
        return reduction + _minimum;
    }
```
If we take less cost making modifier to earlier then most costlier modifier and so on,
What happens is that when a function call came without check it will reverted in first modifier no need to check for others.

For example here, Let say a call came from other than HUB,
so in ```reduceLimitForClone()``` first check happens for noChallenge, noCooldown, alive, then after it will revert as onlyHub failed which is most common and less cost check which happening in the end, And much gas lost in previous checks.

Via reordering we can save those gas of user.
*Instances(1)*
```
File:: contracts/Position.sol
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L97
```

