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

