# Gas Optmization

## Summary

|No  |  issue   | instance  |
|--- |----------|-----------|
|[G‑1]| State variables should be cached in stack variables rather than re-reading them from storage |4
|[G-2]| >= casts less gas then > | 7
|[G-3]| Avoid using state variable in emit | 3
|[G-4]| Use != 0 instead of > 0 for unsigned integer comparison | 5
|[G-5]| Change public state variable visibility to private | 1
|[G-6]|Amounts should be checked for 0 before calling a transfer | 4
|[G-7]| Using calldata instead of memory for read-only arguments in external functions saves gas | 1
|[G-8]|Make 3 event parameters indexed when possible | 3
|[G‑9]| Use nested if statements instead of && | 4






### [G‑1] State variables should be cached in stack variables rather than re-reading them from storage
```solidity
142     zchf.burnFrom(msg.sender, minted - newMinted, reserveContribution);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L142

```solidity
143     minted = newMinted;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L143

```solidity
95      address(zchf),
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L95

```solidity
107     zchf.registerPosition(address(pos));     
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L107

```solidity 
108     zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L108

## [G-2] >= casts less gas then >

```solidity
84     if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();  
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84

```solidity
85     if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();   
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L85

```solidity
104    if (explicit > 0){   
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L104


```solidity
153    if (block.timestamp > minters[_minter]) revert TooLate();    
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L153

```solidity
203    if (challenge.bid > 0) {
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L203

```solidity
267    if (effectiveBid > fundsNeeded){
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L267

```solidity
115     if (amount > 0){
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L115


## [G-3]Avoid using state variable in emit 
 Using a state variable in SetOwner emits wastes gas.

```solidity
69      emit PositionOpened(_owner, original, _zchf, address(collateral), _liqPrice);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L69

```solidity
87      emit PositionOpened(owner, original, address(zchf), address(collateral), _price);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L87

```solidity
287     emit MintingUpdate(collateralBalance(), price, minted, limit);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L87


## [G-4]Use != 0 instead of > 0 for unsigned integer comparison
When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0.

```solidity
381     if (challengedAmount > 0) revert Challenged();
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L381

```solidity
203       if (challenge.bid > 0) {
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L203

```solidity
114     if (amount > 0){
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L114

```solidity
84      if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84


```solidity
85      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L85

## [G-5]Change public state variable visibility to private
 If it is preferred to change the visibility of the owner
 
 ```solidity
 21      address public owner; 
 ```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol



## [G-6]Amounts should be checked for 0 before calling a transfer

```solidity
253     IERC20(token).transfer(target, amount);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L253


```solidity
269     IERC20(collateral).transfer(target, amount);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L269

```solidity
69     chf.transfer(target, amount);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L69

```solidity
280     zchf.transfer(target, proceeds);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L280

## [G-7] Using calldata instead of memory for read-only arguments in external functions saves gas

```solidity
159     Challenge memory copy = Challenge(
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L159



## [G-8] Make 3 event parameters indexed when possible
It's the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
92     event Trade(address who, int amount, uint totPrice, uint newprice); // amount pos or neg for mint or redemption
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L92

```solidity
52    event MinterApplied(address indexed minter, uint256 applicationPeriod, uint256 applicationFee, string message);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L52

```solidity
53    event MinterDenied(address indexed minter, string message);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L53

## [G‑9] Use nested if statements instead of &&

```solidity
85     if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L85

```solidity
86     if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L86

```solidity
267     if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L267

```solidity
294     if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L294






