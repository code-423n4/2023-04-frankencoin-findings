#Using unchecked blocks to save gas
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn't possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol
    156        _balances[sender] -= amount;
    157        _balances[recipient] += amount;
    185        _balances[recipient] += amount;
    203        _totalSupply -= amount;

