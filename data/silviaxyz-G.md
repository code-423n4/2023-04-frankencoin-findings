#  Use Constants as possible

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L98
```solidity
        string public constant NAME = "Frankencoin Pool Share";
    string public constant SYMBOL = "FPS";

        return "Frankencoin Pool Share";
```
```solidity

    function name() override external pure returns (string memory) {
        return NAME;
    }
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L102

```solidity
    string public constant SYMBOL = "FPS";

        return SYMBOL;
```
```solidity
    function symbol() override external pure returns (string memory) {
        return SYMBOL;
    }
```

#  Use ```solidity++i``` instead of  ```solidity++i```. ```solidity++i``` uses less gas

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L192
```solidity
        for (uint i=0; i<helpers.length; i++){
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L196
```solidity
            for (uint j=i+1; j<helpers.length; j++){
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L312
```solidity
        for (uint256 i = 0; i<addressesToWipe.length; i++){
```

# Use custom errors
Use custom errors instead of these inline string errors.

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L81
```solidity
            require(false, "unsupported token");
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L50
```solidity
        require(block.timestamp <= horizon, "expired");
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L51
```solidity
        require(chf.balanceOf(address(this)) <= limit, "limit");
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L242
```solidity
        require(msg.sender == address(zchf), "caller must be zchf");
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L253
```solidity
        require(totalSupply() < 2**128, "total supply exceeded");
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L293
```solidity
        require(shares + ONE_DEC18 < totalShares, "too many shares"); // make sure there is always at least one share
```

# ```x += y``` costs more gas than ```x = x + y``` for state variables
- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L169
```solidity
      minterReserveE6 += _amount * _reservePPM; // minter reserve must be kept accurately in order to ensure we can get back to exactly 
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L196
```solidity
      minterReserveE6 -= amount * reservePPM;
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L227
```solidity
      minterReserveE6 -= targetTotalBurnAmount * _reservePPM; // reduce reserve requirements by original ratio
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L253
```solidity
      minterReserveE6 -= freedAmount * _reservePPM; // reduce reserve requirements by original ratio
```

# Use solidity version 0.8.19 to gain some gas boost
This announcement https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/ suggest that using latest version of solidity compiler could give some gas usage advantages.
- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L4
```solidity
pragma solidity >=0.8.0 <0.9.0;
```
and for other contracts in the project, it is:

```solidity
pragma solidity >=0.8.0 <0.9.0;
```

