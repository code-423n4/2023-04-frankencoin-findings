## GAS-1: <X> <= <Y> costs more gas than <X> < <Y> + 1

### Description

In Solididy, the opcode 'less or equal' doesn't exist. So the EVM will translate this by two distinct operation, first the inferior, and then the equal which cost more gas then a strict less.

### Affected file

* ERC20PermitLight.sol (Line: 30)
* Equity.sol (Line: 136, 244, 247)
* Frankencoin.sol (Line: 141, 282, 294)
* MintingHub.sol (Line: 109, 171, 172, 201, 218, 255)
* Position.sol (Line: 53, 110, 184, 305, 307, 374)
* StablecoinBridge.sol (Line: 50, 51)

## GAS-2: Caching global variables is more expensive than using the actual variable

### Description

 It’s cheaper to use global variable as compared to caching.

### Affected file

* Position.sol (Line: 183)

## GAS-3: Internal functions not called by the contract should be removed to save deployment gas

### Description

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword.

### Affected file

* ERC20.sol (Line: 179, 200)
* MathUtil.sol (Line: 18, 39)

## GAS-4: Replace modifier with function

### Description

Modifiers make code more elegant, but cost more than normal functions.

### Affected file

* Frankencoin.sol (Line: 266)
* MintingHub.sol (Line: 115)
* Ownable.sol (Line: 49)
* Position.sol (Line: 366, 373, 380, 387)

## GAS-5: The result of a function call should be cached rather than re-calling the function

### Description

External calls are expensive. Consider caching.

### Affected file

* Equity.sol (Line: 145, 147)
* Frankencoin.sol (Line: 207, 209)

## GAS-6: Use assembly to check for address(0)

### Description

Saves 6 gas per instance if using assembly to check for address(0).

### Remediation

```
assembly {
 if iszero(_addr) {
  mstore(0x00, "zero address")
  revert(0x00, 0x20)
 }
}
```

### Affected file

* ERC20.sol (Line: 152, 180)
* ERC20PermitLight.sol (Line: 56)

## GAS-7: Using > 0 costs more gas than != 0 when used on a uint in a require() statement

### Description

When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0. This change saves 6 gas per instance.

### Affected file

* Equity.sol (Line: 114)
* Frankencoin.sol (Line: 84, 85, 104)
* MintingHub.sol (Line: 203)
* Position.sol (Line: 381)