## First Vulnerability

Use != 0 instead of > 0 for unsigned integer comparison

## Vulnerability Details

the use of > 0 for unsigned integer comparison costs more gas fees than using != 0 for solidity versions < 0.8.12

## Impact

gas optimization

## Proof of Concept

Instances:6

```solidity
File: contracts/Equity.sol
Line 114:    if (amount > 0){
```

```solidity
File: contracts/Frankencoin.sol
Line 84:     if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
Line 85:     if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
Line 104:    if (explicit > 0){
```

```solidity
File: contracts/MintingHub.sol
Line 203:    if (challenge.bid > 0) {
```

```solidity
File: contracts/Position.sol
Line 203:   if (challengedAmount > 0) revert Challenged();
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Use != 0 instead of > 0 for unsigned integer comparison

#

## Second Vulnerability

Using bool for storing variables incurs overhead.

## Vulnerability Details

Booleans are more expensive than uint256.

## Impact

gas optimization.

## Proof of Concept

Instances:2

```solidity
File: contracts/MathUtil.sol
Line 21:     bool cond;
```

```solidity
File: contracts/ERC20.sol
Line 163:    bool success = transfer(recipient, amount);
```

## Tools Used

Manual Testing.

## Recommended Mitigation Steps

Use uint8(1) and uint8(0) for true/false
