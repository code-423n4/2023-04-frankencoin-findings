| Gas Optimizations List |                                                                                                                   |         |
|------------------------|-------------------------------------------------------------------------------------------------------------------|---------|
| Number                 | Optimization Details                                                                                              | Context |
| [G-01]                 | Use nested if and avoid multiple check combinations                                                               | 4       |
| [G-02]                 | Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct where appropriate | 3       |
| [G-03]                 | Use < (>) instead of <= ( >=)                                                                                     | 2       |
| [G-04]                 | Multiple accesses of a mapping/array should use a local variable cache                                            | 1       |
| [G-05]                 | Upgrade Solidity's optimizer                                                                                          | 1       |

Total 5 issues

###### [G-1] Use nested if and avoid multiple check combinations

Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.
```
contracts\Frankencoin.sol:
84:      if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84>
```
contracts\Frankencoin.sol:
85:      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L85>
```
contracts\Frankencoin.sol:
267:     if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L267>
```
contracts\Position.sol:
294:     if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L294>

###### [G-2] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.
```
contracts\Equity.sol:
83:    mapping (address => address) public delegates;
88:    mapping (address => uint64) private voteAnchor;
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L83>
```
contracts\ERC20.sol:
43:    mapping (address => uint256) private _balances;
45:    mapping (address => mapping (address => uint256)) private _allowances;
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L43>
```
contracts\Frankencoin.sol:
45:    mapping (address => uint256) public minters;
50:    mapping (address => address) public positions;
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L45>

###### [G-3] Use < (>) instead of <= ( >=)

There is no single opcode for ≤ or ≥ in Solidity. Solidity compiler executes LT/GT opcode and then executes an ISZERO opcode to check if the result of the previous comparison (LT/GT) is zero.
2 results - 2 files:
```
contracts\Frankencoin.sol:
141:    if (balance <= minReserve){
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L141>
```
contracts\Frankencoin.sol:
141:    if (block.timestamp <= cooldown) revert Hot();
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L374>

###### [G-4] Multiple accesses of a mapping/array should use a local variable cache

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata

```
contracts\ERC20.sol:
154:       _beforeTokenTransfer(sender, recipient, amount);
155:        if (_balances[sender] < amount) revert ERC20InsufficientBalance(sender,
_balances[sender], amount);
156:        _balances[sender] -= amount;
157:        _balances[recipient] += amount;
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L154-L157>

###### [G-5] Upgrade Solidity's optimizer 

Make sure Solidity’s optimizer is enabled. It reduces gas costs. If you want to gas optimize for contract deployment (costs less to deploy a contract) then set the Solidity optimizer at a low number. If you want to optimize for run-time gas costs (when functions are called on a contract) then set the optimizer to a high number.
Set the optimization value higher than 800 in your hardhat.config.ts file.


```
hardhat.config.ts :
93:  solidity: {
94:      version: "0.8.13",
95:      settings: {
96:          optimizer: {
97:              enabled: true,
98:              runs: 200
99:              },
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/hardhat.config.ts#L93-L99>