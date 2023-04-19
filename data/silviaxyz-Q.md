# To increase readability and maintainability be aware of argument/parameter consistency 
All of the arguments could be like ```limit``` instead of ```_limit``` or ```limit_```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L26
```solidity
    constructor(address other, address zchfAddress, uint256 limit_){
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L83
```solidity
   function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L125
```solidity
   function registerPosition(address _position) override external {
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L152
```solidity
   function denyMinter(address _minter, address[] calldata _helpers, string calldata _message) override external {
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L165
```solidity
   function mint(address _target, uint256 _amount, uint32 _reservePPM, uint32 _feesPPM) override external minterOnly {
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L172
```solidity
   function mint(address _target, uint256 _amount) override external minterOnly {
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L179
```solidity
   function burn(uint256 _amount) external {
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L194
```solidity
   function burn(uint256 amount, uint32 reservePPM) external override minterOnly {
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L204
```solidity
   function calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) public view returns (uint256) {
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L223
```solidity
   function burnFrom(address payer, uint256 targetTotalBurnAmount, uint32 _reservePPM) external override minterOnly returns (uint256) {
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L251
```solidity
   function burnWithReserve(uint256 _amountExcludingReserve, uint32 _reservePPM) external override minterOnly returns (uint256) {
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L262
```solidity
   function burn(address _owner, uint256 _amount) override external minterOnly {
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L280
```solidity
   function notifyLoss(uint256 _amount) override external minterOnly {
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L293
```solidity
   function isMinter(address _minter) override public view returns (bool){
```

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L300
```solidity
   function isPosition(address _position) override public view returns (address){
```

# Contract files should define a locked compiler version
Floating compiler version is not desired.

- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L4
```solidity
 pragma solidity >=0.8.0 <0.9.0;
```

and for other contracts it is:
```solidity
 pragma solidity ^0.8.0;
```

