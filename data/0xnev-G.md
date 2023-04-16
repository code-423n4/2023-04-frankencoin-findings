| |Issue|Instances|Total Gas Saved|
|:-:|:-|:-:|:-:|
| [G-01] | `unchecked` block can be implemented for `ERC20.transferFrom` | 1 |  - |
| [G-02] | `unchecked` block can be implemented for `ERC20._transfer` | 1 |  - |
| [G-03] | `unchecked` block can be implemented for `ERC20._mint` | 1 |  - |
| [G-04] | `unchecked` block can be implemented for `ERC20._burn` | 1 |  - |
| [G-05] | Unchecked blocks when there is previous checks can be implemented | 2 | - |
| [G-06] | State variables can be packed in fewer storage slots | 2 |  15000 |
| [G-07] | Use nested `if` statements to avoid multiple check combinations using `&&` | 4 | 24 |
| [G-08] | `Ownable.onlyOwner()` modifier can be refactored | 1 | - |

| Total Found Issues | 8 |
|:--:|:--:|


### [G-01] `unchecked` block can be implemented for `ERC20.transferFrom`
Here, underflow is not possible due to check of `currentAllowance < amount`, so `_approve` can be wrapped with the `unchecked` block

[ERC20.sol#L132](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L132)
```solidity
function transferFrom(address sender, address recipient, uint256 amount) external override returns (bool) {
    _transfer(sender, recipient, amount);
    uint256 currentAllowance = allowanceInternal(sender, msg.sender);
    if (currentAllowance < INFINITY){
        // Only decrease the allowance if it was not set to 'infinite'
        // Documented in /doc/infiniteallowance.md
        if (currentAllowance < amount) revert ERC20InsufficientAllowance(sender, currentAllowance, amount);
        _approve(sender, msg.sender, currentAllowance - amount);
    }
    return true;
}
```
Gas savings applies to all functions that calls `ERC20.transferFrom()`. For example, saves on average 11 gas for `MintingHub.openPosition` function that calls `ERC20.transferFrom()`. 
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before  | 1586771    | 1611121   | 1615171 | 1615171 |
| After | 1586760    | 1611110  | 1615160 | 1615160 | 


### [G-02] `unchecked` block can be implemented for `ERC20._transfer`
Here, ` _balances[sender] -= amount` will never underflow due to check of `_balances[sender] < amount`.

 `_balances[recipient] += amount` will never overflow since sum of all balances are capped by `totalSupply`.

[ERC20.sol#L156-L157](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L156-L157)
 ```solidity
function _transfer(address sender, address recipient, uint256 amount) internal virtual {
    require(recipient != address(0));
    
    _beforeTokenTransfer(sender, recipient, amount);
    if (_balances[sender] < amount) revert ERC20InsufficientBalance(sender, _balances[sender], amount);
    _balances[sender] -= amount;
    _balances[recipient] += amount;
    emit Transfer(sender, recipient, amount);
}
 ```

Recommendation:
Hence, ` _balances[sender] -= amount` and `_balances[recipient] += amount` can be wrapped with the `unchecked` block.

Gas savings applies to all functions that calls `ERC20._transfer()`. For example, saves on average 238 gas for `Frankencoin.suggestMinter` that calls `ERC20._transfer()`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before  | 28804    | 31804   | 31804 | 34804 |
| After | 28566    | 31566  | 31566 | 34566 |

### [G-03] `unchecked` block can be implemented for `ERC20._mint`
Here, overflow of `_balances[recipient] += amount` is not possible since it is at most `_totalSupply + amount`, which is checked automatically by solidity `^0.8.0`.

[ERC20.sol#L185](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L185)

```solidity
function _mint(address recipient, uint256 amount) internal virtual {
    require(recipient != address(0));

    _beforeTokenTransfer(address(0), recipient, amount);

    _totalSupply += amount;
    _balances[recipient] += amount;
    emit Transfer(address(0), recipient, amount);
}
```

Recommendation:
Hence, `_balances[recipient] += amount` can be wrapped with the `unchecked` block.

Gas savings applies to all functions that calls `ERC20._mint()`. For example, saves on average 242 gas for `Frankencoin.mint(address,uint256,uint32,uint32)  ` that calls `ERC20._mint()`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before  | 7508    | 20048   | 27408 | 31408 |
| After | 7266    | 19806  | 27166 | 31166 |


### [G-04] `unchecked` block can be implemented for `ERC20._burn`
Here, underflow of `_totalSupply -= amount` is not possible since underflow is checked by `_balances[account] -= amount` and `_totalSupply >= _balances[account] >= amount`

[ERC20.sol#L203-L204](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L203-L204)
```solidity
function _burn(address account, uint256 amount) internal virtual {
    _beforeTokenTransfer(account, address(0), amount);

    _totalSupply -= amount;
    _balances[account] -= amount;
    emit Transfer(account, address(0), amount);
}
```
Recommendation:
Hence we can refactor the function such that `_totalSupply -= amount` can be wrapped with the `unchecked` block

```solidity
function _burn(address account, uint256 amount) internal virtual {
    _beforeTokenTransfer(account, address(0), amount);

    _balances[account] -= amount;
    unchecked{
        _totalSupply -= amount;
    }
    emit Transfer(account, address(0), amount);
}
```

Gas savings applies to all functions that calls `ERC20._burn()`. For example, saves on average 82 gas for `Frankencoin.burnFrom()` that calls `ERC20._burn()`

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before  | 8110    | 8110   | 8110 | 8110 |
| After | 8028    | 8028  | 8028 | 8028 |


### [G-05] Unchecked blocks when there is previous checks can be implemented

### Unchecked block for `Position.mintInternal()`
Here, since `limit` is always greater than or equal to `minted + amount`, `minted += amount;` can be wrapped in an `unchecked` block. 

[Position.sol#L196](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L196)
```solidity
function mintInternal(address target, uint256 amount, uint256 collateral_) internal {
    if (minted + amount > limit) revert LimitExceeded();
    zchf.mint(target, amount, reserveContribution, calculateCurrentFee());
    minted += amount;

    checkCollateral(collateral_, price);
    emitUpdate();
}
```

Gas savings applies to all functions that calls `Position.mintInternal()`. For example, saves on average 67 gas for `Position.mint()` function that calls `mintInternal()`
|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before  | 593    | 28287   | 34430 | 58330 |
| After | 593    | 28220  | 34346 | 58246 |



### Unchecked block for `Position.notifyRepaidInternal()`
Here, since `minted` is always greater than or equal to `amount`, `minted -= amount;` can be wrapped in an `unchecked ` block

[Position.sol#L242](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L242)
```solidity
function notifyRepaidInternal(uint256 amount) internal {
    if (amount > minted) revert RepaidTooMuch(amount - minted);
    minted -= amount;
}
```

Gas savings applies to all functions that calls `Position.notifyRepaidInternal()`. For example, saves on average 73 gas for `Position.notifyChallengeSucceeded()` function that calls `Position.notifyRepaidInternal()`.

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before  | 8960   | 10080   | 10080 | 11200 |
| After | 8896    | 10007  | 10007 | 11119 |



### [G-06] State variables can be packed in fewer storage slots
[Position.sol#L38-L39](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L38-L39)
```solidity
2 results - 1 file

/Position.sol
32:    address public immutable original; // originals point to themselves, clone to their origin
33:    address public immutable hub; // the hub this position was created by
34:    IFrankencoin public immutable zchf; // currency
35:    IERC20 public override immutable collateral; // collateral
36:    uint256 public override immutable minimumCollateral; // prevent dust amounts
37:
38:    uint32 public immutable mintingFeePPM; /// @audit 4 bytes can be packed
39:    uint32 public immutable reserveContribution; /// @audit 4 bytes can be packed
```
Each slot of storage can store 32 bytes of data. The first write to an EVM storage position costs 20000. Subsequent writes to an existing storage position costs 5000. Ensure storage packing rules to save gas.

Recommendation:
Saves 25000 - 10000 = 15000 gas
```solidity
address public immutable original; // originals point to themselves, clone to their origin
uint32 public immutable mintingFeePPM;
address public immutable hub; // the hub this position was created by
uint32 public immutable reserveContribution; // in ppm
IFrankencoin public immutable zchf; // currency
IERC20 public override immutable collateral; // collateral
uint256 public override immutable minimumCollateral; // prevent dust amounts
```


### [G-07] Use nested `if` statements to avoid multiple check combinations using `&&`
[Position.sol#L294](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L294)
```solidity
4 results - 2 files

/Position.sol
294:        if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();
```
[Frankencoin.sol#L84-L85](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84-L85)
[Frankencoin.sol#L267](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L267)

```solidity
/Frankencoin.sol
84:      if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();

85:      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();

267:      if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();
```

Using nested `if` statements is cheaper than using `&& ` for multiple check combinations (saves ~6gas). Additionally, it improves readability of code and better coverage reports.


### [G-08] `Ownable.onlyOwner()` modifier can be refactored

`Ownable.requireOwner()` can be removed by refactoring `onlyOwner()` modifier.

[Ownable.sol#L45-L52](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L45-L52)
```solidity
function requireOwner(address sender) internal view {
    if (owner != sender) revert NotOwner();
}

modifier onlyOwner() {
    requireOwner(msg.sender);
    _;
}
```

Recommendation:
```solidity
modifier onlyOwner() {
    if (owner != msg.sender) revert NotOwner();
    _;
}
```
Gas savings applies to all functions that uses the `onlyOwner()` modifier. For example saves on average 76 gas for `Position.adjust()` function that uses `onlyOwner()` modifier.

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before  | 9963   | 16336   | 9963 | 39828 |
| After | 9887    | 16263  | 9887 | 39752 |


