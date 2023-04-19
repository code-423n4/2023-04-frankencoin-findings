# Low Risk

|#|Title|
|---|---|
|L-1|Missing checks for `address(0x0)` in ERC20.sol|
|L-2|Redundant casting|
|L-3|Validation should be at the top of the function|

## [L-1] Missing checks for `address(0x0)` in ERC20.sol

According to the description of the function, 'sender' cannot be the zero address, validation is not implemented.

```solidity
File: contracts/ERC20.sol
136: 
137:     /**
138:      * @dev Moves tokens `amount` from `sender` to `recipient`.
139:      *
140:      * This is internal function is equivalent to `transfer`, and can be used to
141:      * e.g. implement automatic token fees, slashing mechanisms, etc.
142:      *
143:      * Emits a `Transfer` event.
144:      *
145:      * Requirements:
146:      *
147:      * - `sender` cannot be the zero address.
148:      * - `recipient` cannot be the zero address.
149:      * - `sender` must have a balance of at least `amount`.
150:      */
151:     function _transfer(address sender, address recipient, uint256 amount) internal virtual {
152:         require(recipient != address(0));
153: 
154:         _beforeTokenTransfer(sender, recipient, amount);
155:         if (_balances[sender] < amount) revert ERC20InsufficientBalance(sender, _balances[sender], amount);
156:         _balances[sender] -= amount;
157:         _balances[recipient] += amount;
158:         emit Transfer(sender, recipient, amount);
159:     }
```

## [L-2] Redundant casting
IPosition is redundant, factory return address and only address is needed in functions.
```diff
--- a/original.md
+++ b/modified.md
@@ -3,7 +3,7 @@ File: contracts/MintingHub.sol
 089:         address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
 090:         uint256 _mintingMaximum, uint256 _initPeriodSeconds, uint256 _expirationSeconds, uint256 _challengeSeconds,
 091:         uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address) {
-092:         IPosition pos = IPosition(
+092:         address pos =
 093:             POSITION_FACTORY.createNewPosition(
 094:                 msg.sender,
 095:                 address(zchf),
@@ -16,12 +16,12 @@ File: contracts/MintingHub.sol
 102:                 _mintingFeePPM,
 103:                 _liqPrice,
 104:                 _reservePPM
-105:             )
+105: 
 106:         );
-107:         zchf.registerPosition(address(pos));
+107:         zchf.registerPosition(pos);
 108:         zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE);
 109:         require(_initialCollateral >= _minCollateral, "must start with min col");
-110:         IERC20(_collateralAddress).transferFrom(msg.sender, address(pos), _initialCollateral);
+110:         IERC20(_collateralAddress).transferFrom(msg.sender, pos, _initialCollateral);
 111: 
-112:         return address(pos);
+112:         return pos;
 113:     }
 ```
 
`pos` casting to `address` is redundant, it is already `address` type.
```solidity
File: contracts/MintingHub.sol
127:         address pos = POSITION_FACTORY.clonePosition(position);
128:         zchf.registerPosition(pos);
129:         existing.collateral().transferFrom(msg.sender, address(pos), _initialCollateral); // audit:low redundant casting
130:         IPosition(pos).initializeClone(msg.sender, existing.price(), limit, _initialCollateral, _initialMint);
131:         return address(pos); // audit:low redundant casting
```

`position.collateral()` already returns `IERC20`
```solidity
File: contracts/MintingHub.sol
142:         IERC20(position.collateral()).transferFrom(msg.sender, address(this), _collateralAmount); //audit:low redundant casting
```

`zchf` is already `IFrankencoin` extended from `IERC20`
```solidity
File: contracts/MintingHub.sol
263:             IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid); //audit:low redundant casting
```

`zchf.reserve()` already returns `IReserve`
```solidity
File: contracts/Position.sol
111:         IReserve(zchf.reserve()).checkQualified(msg.sender, helpers); //audit:low redundant casting
```

`collateral` is already `IERC20`
```solidity
File: contracts/Position.sol
170:         return IERC20(collateral).balanceOf(address(this)); //audit:low redundant casting
```

`zchf` is already `IFrankencoin` extended from `IERC20`
```solidity
File: contracts/Position.sol
228:         IERC20(zchf).transferFrom(msg.sender, address(this), amount); //audit:low redundant casting
```

`zchf` is already `IFrankencoin`
```solidity
File: contracts/Position.sol
233:         uint256 actuallyBurned = IFrankencoin(zchf).burnWithReserve(burnable, reserveContribution); //audit:low redundant casting
```

`collateral` is already `IERC20`
```solidity
File: contracts/Position.sol
269:         IERC20(collateral).transfer(target, amount); //audit:low redundant casting
```

`hub` is already `address` type
```solidity
File: contracts/Position.sol
388:         if (msg.sender != address(hub)) revert NotHub(); //audit:low redundant casting
```
## [L-3] Validation should be at the top of the function

```solidity
File: contracts/MintingHub.sol
109:         require(_initialCollateral >= _minCollateral, "must start with min col");
```

# Non-Critical Risk

|#|Title|
|---|---|
|N-1|Typos|
|N-2|Consider changing the name to a more descriptive|
|N-3|Function ordering does not follow the Solidity style guide|

## [N-1] Typos
Consider using the [VSCode plugin](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker) or similar to help catch spelling errors during development.
```solidity
proporational => proportional
File: contracts/Equity.sol 
34:      * I.e., the supply is proporational to the cubic root of the reserve and the price is proportional to the

inflaction => inflation
File: contracts/Equity.sol
73:      * cap of 3,000,000,000,000,000,000 CHF. This limit could in theory be reached in times of hyper inflation.

responsiblity => responsibility
helpes => helps
File: contracts/Equity.sol
206:      * directly or indirectly to the sender. It is the responsibility of the caller to figure out whether
207:      * helpes are necessary and to identify them by scanning the blockchain for Delegation events.

transfered => transferred
File: contracts/MintingHub.sol
247:      * In case that the collateral cannot be transferred back to the challenger (i.e. because the collateral token has a blacklist and the

creterion => criterion
File: contracts/Position.sol
358:      * This is also a good creterion when deciding whether it should be shown in a frontend.


```

## [N-2] Consider changing the name to a more descriptive
A good variable name can replace documentation, which improves code readability.
```solidity
File: contracts/Position.sol
136:         uint256 colbal = collateralBalance(); //audit:non
```

```solidity
File: contracts/Position.sol
331:         uint256 colBal = collateralBalance(); //audit:non
```

## [N-3] Function ordering does not follow the Solidity style guide
According to the [Solidity style guide](https://docs.soliditylang.org/en/v0.8.13/style-guide.html#order-of-functions), functions should be laid out in the following order:
- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private