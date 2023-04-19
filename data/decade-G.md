# Gas Optimizations
## 1. `totalSupply()` unnecessarily checked twice. 
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L83-L85
```
   function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {
      if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
```
- In a if statement with `&&` operand, if the first check is fails, second is check is not reached and if statement returns false. Hence, putting `totalSupply()>0` check first will not require you to put it twice. 

---
## 2. `_minCollateral` require check in function `openPosition()` can be shifted above so that users have to less fee on **transaction failure**. 

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L88-L113
 - In the `openPosition()` function, it first deploys a position contract and then checks the below require statement. 
 ```
    require(_initialCollateral >= _minCollateral, "must start with min col");
 ```
 - If this require statement fails, then transaction fails. But user still have to pay the gas used by the transaction upto that require statement. 
 - Now, since the position contract is deployed, this gas used fee also increases multifold since the transaction inevitably fails. 
 - If this require check is placed before the position contract is created, then user would not have to pay that much amount during transaction. 
 
---

## 3. No need wrapping `createClone()` return value inside `Position` interface in `clonePosition()` function. 
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L30
```
    function clonePosition(address _existing) external returns (address) {
        Position existing = Position(_existing);
        Position clone = Position(createClone(existing.original()));
        return address(clone);
    }
```
- `createClone()` returns address itself. There is not need for wrapping it under `Position` interface and then return `address(clone)`. You can directly return the address. 

