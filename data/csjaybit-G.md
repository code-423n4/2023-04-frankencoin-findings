[G-1] Use nested if and, avoid multiple check combinations

Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

```solidity
File: contracts/Position.sol
        if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();
```
```solidity 
file: contracts/Frankencoin.sol
 
      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();\\line-85

      //line 266-269
   modifier minterOnly() {
      if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();
      _;
   } 
    //line 294
  return minters[_minter] != 0 && block.timestamp >= minters[_minter];
```

