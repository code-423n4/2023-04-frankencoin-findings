# Gas Optimisations:
## Multiple mappings related to the same data can be combined into a single mapping using a struct:
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.
**6 Instances** of this issue:
contracts/Equity.sol#L83-88
```
mapping (address => address) public delegates;
mapping (address => uint64) private voteAnchor;
```
can be refactored to:
```
struct Params {
	address delegates;
	uint64 voteAnchor;
}
mapping (address => Params) public params;
```
contracts/Frankencoin.sol#L45-L50
```
mapping (address => uint256) public minters;
mapping (address => address) public positions;
```
can be refactored to:
```
struct Params {
	uint256 minters;
	address positions;
}
mapping (address => Params) public params;
```
contracts/ERC20.sol#L43-L45
```
mapping (address => uint256) private _balances;
mapping (address => mapping (address => uint256)) private _allowances;
```
can be refactored to:
```
struct Params {
	address _balances;
	mapping (address => uint256) _allowances;
}
mapping (address => Params) private params;
```

## Avoid compound assignment operator in state variables
Using compound assignment operators for state variables (like State += X or State -= X ...) it's more expensive than using operator assignment (like State = State + X or State = State - X ...).
saves **~13 gas per entry (without optimizitions)**
change 
`minterReserveE6 += _amount * _reservePPM;` 
to
` minterReserveE6 = minterReserveE6 + (_amount * _reservePPM);`
**11 Instances**:
```
contracts/Frankencoin.sol#169 minterReserveE6 += _amount * _reservePPM;
contracts/Frankencoin.sol#196 minterReserveE6 -= amount * reservePPM;
contracts/Frankencoin.sol#227 minterReserveE6 -= targetTotalBurnAmount * _reservePPM;
contracts/Frankencoin.sol#253 minterReserveE6 -= freedAmount * _reservePPM;
contracts/ERC20.sol#L184 _totalSupply += amount;
contracts/ERC20.sol#L203 _totalSupply -= amount;
contracts/Position.sol#L196 minted += amount;
contracts/Position.sol#L242 minted -= amount;
contracts/Position.sol#L295 challengedAmount += size;
contracts/Position.sol#L309 challengedAmount -= _collateralAmount;
contracts/Position.sol#L330 challengedAmount -= _size;
```
 

## Can Avoid setting DOMAIN_SEPARATOR every time the functions are called.
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L37
At permit function, it sets DOMAIN_SEPARATOR every time the function is called. DOMAIN_SEPARATOR can be set beforehand.