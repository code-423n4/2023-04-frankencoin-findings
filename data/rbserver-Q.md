## QA REPORT

| |Issue|
|-|:-|
| [01] | COLLECTING OPENING FEE WHEN OPENING A POSITION CAN BE UNFAIR |
| [02] | `INFINITY` IS NOT `type(uint256).max` |
| [03] | `shares` CANNOT BE UP TO `totalShares - ONE_DEC18` IN `Equity.calculateProceeds` FUNCTION |
| [04] | USING `uint256` IN `Equity.anchorTime` FUNCTION CAN BE MORE FUTURE-PROOFED |
| [05] | REDUNDANT CAST |
| [06] | UNUSED IMPORTS |
| [07] | IMMUTABLES CAN USE SAME NAMING CONVENTION |
| [08] | `type(uint128).max` CAN BE USED IN `Equity.onTokenTransfer` FUNCTION'S `require` STATEMENT |
| [09] | WORD TYPING TYPO |
| [10] | `1000_000` CAN BE CODED AS `1_000_000` IN `Frankencoin.mint` FUNCTION |

## [01] COLLECTING OPENING FEE WHEN OPENING A POSITION CAN BE UNFAIR
The following `Position.deny` function can be called to immediately expire a freshly proposed position for any reason. The opened position's owner has to pay an opening fee but always faces the risk of having the opened position expired for any reason even though such owner would think that she or he opened a legit position. If the position is indeed legit but is denied, the user essentially paid and lost the opening fee for nothing. To be more fair to such owners and also to encourage users from opening positions, please consider making the owners to pay the opening fee when starting to mint after the initialization period instead of when opening the positions.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L109-L114
```solidity
    function deny(address[] calldata helpers, string calldata message) public {
        if (block.timestamp >= start) revert TooLate();
        IReserve(zchf.reserve()).checkQualified(msg.sender, helpers);
        cooldown = expiration; // since expiration is immutable, we put it under cooldown until the end
        emit PositionDenied(msg.sender, message);
    }
```

## [02] `INFINITY` IS NOT `type(uint256).max`
The following `INFINITY` is set to `(1 << 255)`, which is not `type(uint256).max`, even though `INFINITY` is an `uint256`. This is unlike many other protocols' common practice; for example, as shown below, USDT sets  `MAX_UINT` to `2**256 - 1`. Hence, users, who are familar with such common practice, can assume that this protocol's `INFINITY` is also `type(uint256).max` and would expect the `ERC20.transferFrom` function below to decrease the allowance that was set to an amount that is less than `type(uint256).max`. Yet, if such allowance was set to an amount that is less than `type(uint256).max` but more than `(1 << 255)`, the allowance will not be decreased when calling the `ERC20.transferFrom` function below, which can result in unexpectedness when user expects the allowance to decrease but it does not. To avoid such unexpectedness, please consider updating `INFINITY` to `type(uint256).max`.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L47
```solidity
    uint256 internal constant INFINITY = (1 << 255);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L125-L135
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

https://etherscan.io/token/0xdac17f958d2ee523a2206206994597c13d831ec7#code#L163
```solidity
    uint public constant MAX_UINT = 2**256 - 1;
```

## [03] `shares` CANNOT BE UP TO `totalShares - ONE_DEC18` IN `Equity.calculateProceeds` FUNCTION
Although the following `Equity.calculateProceeds` function's comment states: `make sure there is always at least one share`, the `shares` input cannot be up to `totalShares - ONE_DEC18`. It can only be up to `totalShares - ONE_DEC18 - 1` to satisfy `require(shares + ONE_DEC18 < totalShares, "too many shares")`. If the comment is correct, this `require` statement should be updated to `require(shares + ONE_DEC18 <= totalShares, "too many shares")`. Otherwise, the comment can be updated to `make sure there is always at least (ONE_DEC18 + 1) wei shares` to be more accurate.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L290-L297
```solidity
    function calculateProceeds(uint256 shares) public view returns (uint256) {
        uint256 totalShares = totalSupply();
        uint256 capital = zchf.equity();
        require(shares + ONE_DEC18 < totalShares, "too many shares"); // make sure there is always at least one share
        uint256 newTotalShares = totalShares - shares;
        uint256 newCapital = _mulD18(capital, _power3(_divD18(newTotalShares, totalShares)));
        return capital - newCapital;
    }
```

## [04] USING `uint256` IN `Equity.anchorTime` FUNCTION CAN BE MORE FUTURE-PROOFED
`block.number` always increases and can grow even faster especially when the chain becomes more efficient. It is possible that `block.number << BLOCK_TIME_RESOLUTION_BITS` can become too large for `uint64` to hold in the future. If this happens, the following `Equity.anchorTime` function will return an incorrect value that causes many calculations that rely on `anchorTime()` to be incorrect. To be more future-proofed, please consider using `uint256` instead of `uint64` in this function.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L172-L174
```solidity
    function anchorTime() internal view returns (uint64){
        return uint64(block.number << BLOCK_TIME_RESOLUTION_BITS);
    }
```

# [05] REDUNDANT CAST
The following `PositionFactory.clonePosition` function executes `Position clone = Position(createClone(existing.original()))` and then return `address(clone)`. However, `createClone(existing.original())` is already an `address` so there is no need to cast it to `Position` and then return its address. Please consider directly return `createClone(existing.original())` for higher efficiency.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L30-L34
```solidity
    function clonePosition(address _existing) external returns (address) {
        Position existing = Position(_existing);
        Position clone = Position(createClone(existing.original()));
        return address(clone);
    }
```

# [06] UNUSED IMPORTS
The `IReserve.sol` and `Ownable.sol` are not used in the `MintingHub` contract, please consider removing them for better code efficiency.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L5-L7
```solidity
import "./IReserve.sol";
...
import "./Ownable.sol";
```


# [07] IMMUTABLES CAN USE SAME NAMING CONVENTION
As shown below, some immutables are named using capital letters and underscores while the other immutables are named using lowercased letters. To be more consistent, please consider using the same naming convention for all immutables.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L26-L31
```solidity
   uint256 public immutable MIN_APPLICATION_PERIOD; // for example 10 days
   ...
   IReserve override public immutable reserve;
```

# [08] `type(uint128).max` CAN BE USED IN `Equity.onTokenTransfer` FUNCTION'S `require` STATEMENT
The following `Equity.onTokenTransfer` function executes `require(totalSupply() < 2**128, "total supply exceeded")`. Make the code more readable, please consider updating this `require` statement to `require(totalSupply() <= type(uint128).max, "total supply exceeded")`.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L241-L255
```solidity
    function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool) {
        ...
        // the 128 bits are 68 bits for magnitude and 60 bits for precision, as calculated in an above comment
        require(totalSupply() < 2**128, "total supply exceeded");
        return true;
    }
```

# [09] `1000_000` CAN BE CODED AS `1_000_000` IN `Frankencoin.mint` FUNCTION
It is a common practice to separate each 3 digits in a number by an underscore. The `1000_000` used in the `Frankencoin.mint` function below can be coded as  `1_000_000` to improve the code readability.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L165-L170
```solidity
   function mint(address _target, uint256 _amount, uint32 _reservePPM, uint32 _feesPPM) override external minterOnly {
      uint256 usableMint = (_amount * (1000_000 - _feesPPM - _reservePPM)) / 1000_000; // rounding down is fine
      _mint(_target, usableMint);
      _mint(address(reserve), _amount - usableMint); // rest goes to equity as reserves or as fees
      minterReserveE6 += _amount * _reservePPM; // minter reserve must be kept accurately in order to ensure we can get back to exactly 0
   }
```

## [10] WORD TYPING TYPO
The following comment states: `This limit could in theory be reached in times of hyper inflaction`, where `inflaction` is mistyped. Please change `inflaction` to `inflation`.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L63-L74
```solidity
    /**
     ...
     * cap of 3,000,000,000,000,000,000 CHF. This limit could in theory be reached in times of hyper inflaction. 
     */
```