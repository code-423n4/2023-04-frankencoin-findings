# Use of Bitwise Operator Shift left instead of Multiplication and Exponent Operator wherever possible

I found that in the Automated Findings (Gas - 12) https://gist.github.com/CodingNameKiki/36f3bfb214907d68fdf3a43cb0cb8ae3#GAS%E2%80%9112 It was able to detect one of the places where gas optimization was possible, however it missed other places similar to it.


Context: Bitwise operator `Shift left (<<)` should be used for Multiplication by 2 and Powers of 2 because shift left operator is gas efficient as compared to `multiplication (*)` and `exponent (**)` operator. I found this reference of operators and their gas requirement: https://www.evm.codes/?fork=shanghai

I found below places other than mentioned in Automated findings where gas optimization can be done: 

1. Link: https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L25

`
    File: contracts/MathUtil.sol

    Current State:
    150: x = _mulD18(x, _divD18( (powX3 + 2 * _v) , (2 * powX3 + _v)));
    
    Proposed Change:
    150:  x = _mulD18(x, _divD18( (powX3 + (_v << 1)) , ((powX3 << 1) + _v)));
`

2. Link: https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L253

`
    File: contracts/Equity.sol

    Current State:
    253: require(totalSupply() < 2**128, "total supply exceeded");
    
    Proposed Change:
    253: require(totalSupply() < (1<<128), "total supply exceeded");
`
