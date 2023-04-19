# QA Report

## `INFINITY` value is not set to max

`ERC20.sol` allows for infinite approval, and sets `INFINITY` as follows:

```solidity
    uint256 internal constant INFINITY = (1 << 255);
```

Convention would be to set this value to the maximum uint256 value `type(uint256).max == 2 ** 256 - 1`, however here it is equal to `1 << 255` which is half the max uint256 value.

## Insufficient Coverage

The test coverage rate of the project is only 98%. Testing all functions is best practice in terms of security criteria.

Due to its capacity, test coverage is expected to be 100%.

## Missing events

The `StablecoinBridge` contract ought to emit events within its `mintInternal` and `burnInternal` functions. Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

## Function writing that does not comply with the Solidity Style Guide

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

Instances:
- Frankencoin.sol
- Equity.sol
- Position.sol

## `ERC20::transferFrom` ought to be marked `virtual`

In `ERC20`, `transfer` is marked as `virtual` whereas `transferFrom` is not. For consistency, and to adhere to ERC20 common practice, mark `transferFrom` as `virtual`.