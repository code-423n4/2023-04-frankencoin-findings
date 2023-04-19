# Gas-01 improve logic of the `suggestMinter` function

Improve the gas cost by changing the logic of the function `suggestMinter`. The related code snippet is as follow. 

```solidity
File: Frankencoin.sol
83:    function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {
84:       if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
85:       if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow(); // @audit-info  after the totalsupply is not 0, the application fee must be greater than the minimum fee to pass the check. So it takes fee wheen you want a new bridge
86:       if (minters[_minter] != 0) revert AlreadyRegistered(); 
87:       if (totalSupply() > 0)  _transfer(msg.sender, address(reserve), _applicationFee); // @audit low-risk change to this because it avoids someone to send the fee mistakenly to the contract even wehn the total supply is 0
88:       minters[_minter] = block.timestamp + _applicationPeriod;
89:       emit MinterApplied(_minter, _applicationPeriod, _applicationFee, _message);  // @audit missing the msg.sender as an very important parameter to log the event
90:    }
```

## Proof of Concept

This is the gas cost before and after the change for the `suggestMinter` function when run the hardhat test. 

Before:

```diff
|  Frankencoin       ·  suggestMinter             ·      49958  ·      79972  ·      57726  ·            8  ·          -  │
```

After: 

```
|  Frankencoin       ·  suggestMinter             ·      49614  ·      79932  ·      57458  ·            8  ·          -  │
```

## Recommendation

Change the function as follow: 

```diff
function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {
+       if (minters[_minter] != 0) revert AlreadyRegistered(); // put this at the beggining of the function
+       if (totalSupply() > 0) {
+          if (_applicationPeriod < MIN_APPLICATION_PERIOD) revert PeriodTooShort();
+          if (_applicationFee < MIN_FEE) revert FeeTooLow();
+          _transfer(msg.sender, address(reserve), _applicationFee); 
+      }
-       if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
-       if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow(); // @audit-info  after the totalsupply is not 0, the application fee must be greater than the minimum fee to pass the check. So it takes fee wheen you want a new bridge
-       if (minters[_minter] != 0) revert AlreadyRegistered(); 
-       if (totalSupply() > 0)  _transfer(msg.sender, address(reserve), _applicationFee); // @audit low-risk change to this because it avoids someone to send the fee mistakenly to the contract even wehn the total supply is 0
      minters[_minter] = block.timestamp + _applicationPeriod;
      emit MinterApplied(_minter, _applicationPeriod, _applicationFee, _message);  // @audit missing the msg.sender as an very important parameter to log the event
   }
```

This can optimize the logic of the function and it saves gas.