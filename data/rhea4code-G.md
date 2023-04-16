# 1. Don't Initialize Variables with Default Value

## Description
Uninitialized variables are assigned with the types default value.If a variable is not set/initialized, it is assumed to have the default value (0, false, 0x0 etc depending on the data type). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

## Proof of Concept
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L192
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L312

# 2. Cache Array Length Outside of Loop
## Description
Uninitialized variables are assigned with the types default value.If a variable is not set/initialized, it is assumed to have the default value (0, false, 0x0 etc depending on the data type). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

## Proof of Concept
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L192
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L196
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L312
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L143
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L174

# 3. Use != 0 instead of > 0 for Unsigned Integer Comparison
## Description
When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0. The comparison will be true because unsigned integers will always be non-negative.

## Proof of Concept
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L114
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L85
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L104
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L203
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L381

# 4. Long Revert Strings
## Description
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using [Custom Errors](https://blog.soliditylang.org/2021/04/21/custom-errors/) as they are more gas efficient while allowing developers to describe the error in detail using [NatSpec](https://docs.soliditylang.org/en/latest/natspec-format.html).

## Proof of Concept
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L40
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L65

# 5. Use Shift Right/Left instead of Division/Multiplication if possible
## Description
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using [Custom Errors](https://blog.soliditylang.org/2021/04/21/custom-errors/) as they are more gas efficient while allowing developers to describe the error in detail using [NatSpec](https://docs.soliditylang.org/en/latest/natspec-format.html).

## Proof of Concept
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L25
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L53
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L76
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L88
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L305
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L57
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L235
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L237
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L239
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L247
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L26
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L98