# Wrong comment

Comment in Frenkencoin.sol#L77-81
> The caller must assume that someone will veto the new minter unless there is broad consensus that the new minter adds value to the Frankencoin system. Complex proposals should have **application periods** and **applications fees** **above the minimum**. It is assumed that over time, informal ways to coordinate on new minters emerge. The message parameter might be useful for initiating further communication. Maybe it contains a link to a website describing the proposed minter.

If we look at the code this is not the case

```solidity
    Frankencoin.sol
    
    84: if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
    85: if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
```

If the *_applicationPeriod* is above or equal to the *MIN_APPLICATION_PERIOD* this check will pass. So the comment should state ( same goes for *_applicationFee* )

> Complex proposals should have **application periods** and **applications fees** **above or equal to the  the minimum**.

Or change the requirement to 
```solidity
    Frankencoin.sol
    
    84: if (_applicationPeriod <= MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
    85: if (_applicationFee <= MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
```

# Missing require message
### ERC20.sol#L3-10

```solidity
// Adjustments:
// - modifications to support ERC-677
// - removed require messages to save space
// - removed unnecessary require statements
// - removed GSN Context
// - upgraded to 0.8 to drop SafeMath
// - let name() and symbol() be implemented by subclass
// - infinite allowance support, with 2^255 and above considered infinite
```

This comment says that require messages are removed, but they are good in case of a revert because they can help fix the errors easier.

## Instances
### ERC20.sol
```solidity
153: require(recipient != address(0));
182: require(recipient != address(0));
```
# Use 1eN instead of 10**N and number literals


## Instances
### Frankencoin.sol

```solidity
-    uint256 public constant MIN_FEE = 1000 * (10**18);
+    uint256 public constant MIN_FEE = 1000 * (1e18);
```

```solidity
    function minterReserve() public view returns (uint256) {
-      return minterReserveE6 / 1000000;
+      return minterReserveE6 / 1e6; // or even better add a constant because 1e16 is used a lot
    }
```
```solidity
   function calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) public view returns (uint256) {
-    uint256 theoreticalReserve = _reservePPM * mintedAmount / 1e6;
+    uint256 theoreticalReserve = _reservePPM * mintedAmount / 1e6;
     ...
   }
```

### MintingHub.sol

```solidity
-    uint256 public constant OPENING_FEE = 1000 * 10**18;
+    uint256 public constant OPENING_FEE = 1000 * 10e18;
```

### MathUtil.sol

```solidity
-    uint256 internal constant ONE_DEC18 = 10**18;
+    uint256 internal constant ONE_DEC18 = 10e18;
```

```solidity
-    uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01
+    uint256 internal constant THRESH_DEC18 =  1e16
```

# Misleading function name
### Frankencoin.sol

By the name one would think it returns bool true or false if position _position is indeed in the mapping but instead returns the minter of the position. More suitable name would be **positionMinter(address _position)**. ( or something similar)

```solidity
   /**
    * Returns the address of the minter that created this position or null if the provided address is unknown.
    */
   function isPosition(address _position) override public view returns (address){
      return positions[_position];
   }
```

# Misnamed parameters in interface

Implementation uses **feesPPM** where interface uses **feePPM**. Also underscores are missing.

### IFrankencoin.sol
```solidity
        function mint(address target, uint256 amount, uint32 reservePPM, uint32 feePPM) external;

```

### Frankencoin.sol
```solidity
       function mint(address _target, uint256 _amount, uint32 _reservePPM, uint32 _feesPPM) override external minterOnly {
```

# Missing address(0) check

Some functions are missing check for address(0) which could lead to inproper initialization or unexpected outcomes.

### Instances

### Ownable.sol
Could transfer ownership of *Position* to address(0)

```solidity
    function transferOwnership(address newOwner) public onlyOwner {
+       require(newOwner != address(0));
        setOwner(newOwner);
    }

    /**
     * @dev Transfers ownership of the contract to a new account (`newOwner`).
     * Internal function without access restriction.
     */
    function setOwner(address newOwner) internal {
        address oldOwner = owner;
        owner = newOwner;
        emit OwnershipTransferred(oldOwner, newOwner);
    }
```

### StablecoinBridge.sol
No check if *other* or *zchfAddress* is address(0). If is set to 0 by mistake the whole contract must be redeployed since both variables are defined as *immutable*.
```solidity
    constructor(address other, address zchfAddress, uint256 limit_){
        chf = IERC20(other);
        zchf = IFrankencoin(zchfAddress);
        horizon = block.timestamp + 52 weeks;
        limit = limit_;
    }
```

### StablecoinBridge.sol
Same as with StabelcoinBridge. No check for address(0) for immutable variable *zchf*.

```solidity
    constructor(address _zchf, address factory) {
        zchf = IFrankencoin(_zchf);
        POSITION_FACTORY = IPositionFactory(factory);
    }
```

# Style of some functions does not follow the Solidity style guide
### Instances
### IFrankenCoin.sol
```solidity
    function suggestMinter(address _minter, uint256 _applicationPeriod, 
      uint256 _applicationFee, string calldata _message) external;
```

but should be
```solidity
    function suggestMinter(
        address _minter, 
        uint256 _applicationPeriod, 
        uint256 _applicationFee, 
        string calldata _message
    ) external;
```

### MintingHub.sol
```solidity
    function openPosition(
        address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
        uint256 _mintingMaximum, uint256 _expirationSeconds, uint256 _challengeSeconds,
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address) {
            ...
    }
```

but should be 

```solidity
    function openPosition(
        address _collateralAddress, 
        uint256 _minCollateral, 
        uint256 _initialCollateral,
        uint256 _mintingMaximum, 
        uint256 _expirationSeconds, 
        uint256 _challengeSeconds,
        uint32 _mintingFeePPM, 
        uint256 _liqPrice, 
        uint32 _reservePPM
    ) 
        public 
        returns (address) {
            ...
    }
```

Same goes for the other openPosition function in the MintingHub.sol
```solidity
function openPosition(
        address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
        uint256 _mintingMaximum, uint256 _initPeriodSeconds, uint256 _expirationSeconds, uint256 _challengeSeconds,
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address) {
            ...
        }
```

but should be

```solidity
    function openPosition(
        address _collateralAddress, 
        uint256 _minCollateral, 
        uint256 _initialCollateral,
        uint256 _mintingMaximum, 
        uint256 _initPeriodSeconds, 
        uint256 _expirationSeconds, 
        uint256 _challengeSeconds,
        uint32 _mintingFeePPM, 
        uint256 _liqPrice, 
        uint32 _reservePPM
    ) 
        public 
        returns (address) 
    {
            ...
    }
```

### Position.sol
```solidity
    constructor(address _owner, address _hub, address _zchf, address _collateral, 
        uint256 _minCollateral, uint256 _initialLimit, uint256 initPeriod, uint256 _duration,
        uint256 _challengePeriod, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) {
        ...
    }
```

but should be 

```solidity
    constructor(
        address _owner, 
        address _hub, 
        address _zchf, 
        address _collateral, 
        uint256 _minCollateral, 
        uint256 _initialLimit, 
        uint256 initPeriod, 
        uint256 _duration,
        uint256 _challengePeriod, 
        uint32 _mintingFeePPM, 
        uint256 _liqPrice, 
        uint32 _reservePPM) 
    {
        ...
    }
```

### PositionFactory.sol
```solidity
    function createNewPosition(address _owner, address _zchf, address _collateral, uint256 _minCollateral, 
        uint256 _initialLimit, uint256 initPeriod, uint256 _duration, uint256 _challengePeriod, 
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reserve) 
        external returns (address) {
        ...
    }
```

but should be

```solidity
    function createNewPosition(
        address _owner, 
        address _zchf, 
        address _collateral, 
        uint256 _minCollateral, 
        uint256 _initialLimit, 
        uint256 initPeriod, 
        uint256 _duration, 
        uint256 _challengePeriod, 
        uint32 _mintingFeePPM, 
        uint256 _liqPrice, 
        uint32 _reserve
    ) 
        external 
        returns (address) 
    {
        ...
    }
```


# Contract exceeds maximum line length
According to Solidity style guide line length should not be longer than 120 characters.


# Pack variables
Packing the following variables in this way will use 1 storage slot less
```solidity
	uint256 private constant MINIMUM_EQUITY
	uint256 public constant MIN_HOLDING_DURATION = 90*7200 << BLOCK_TIME_RESOLUTION_BITS; // Set to 5 for local testing
	uint192 private totalVotesAtAnchor;  // Total number of votes at the anchor time, see comment on the um
	uint32 private constant QUORUM = 300;
	uint32 public constant VALUATION_FACTOR = 3;
    uint64 private totalVotesAnchorTime; // 40 Bit for the block number, 24 Bit sub-block time resolution
	uint8 private constant BLOCK_TIME_RESOLUTION_BITS = 24;
```

# Incorrect order of funtions
Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.

Functions should be grouped according to their visibility and ordered:

> constructor
>
>receive function (if exists)
>
>fallback function (if exists)
>
>external
>
>public
>
>internal
>
>private

```solidity
contract A {
    constructor() {
        // ...
    }

    receive() external payable {
        // ...
    }

    fallback() external {
        // ...
    }

    // External functions
    // ...

    // External functions that are view
    // ...

    // External functions that are pure
    // ...

    // Public functions
    // ...

    // Internal functions
    // ...

    // Private functions
    // ...
}
```

# Checking for allowance after deploying a contract
Thew following function in **MintingHub.sol** first deploys a new position and then tries to call **transferFrom**. This can waste a lot of gas for user is user didn't already approve the contract to spend the **ZCHF** tokens. Since contract deployment are costly this can quickly add up.

```solidity
 function openPosition(
        address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
        uint256 _mintingMaximum, uint256 _initPeriodSeconds, uint256 _expirationSeconds, uint256 _challengeSeconds,
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address) {
        IPosition pos = IPosition(
            POSITION_FACTORY.createNewPosition(
                msg.sender,
                address(zchf),
                _collateralAddress,
                _minCollateral,
                _mintingMaximum,
                _initPeriodSeconds,
                _expirationSeconds,
                _challengeSeconds,
                _mintingFeePPM,
                _liqPrice,
                _reservePPM
            )
        );
        zchf.registerPosition(address(pos));
        
        // this call includes check for allowance
        zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE); 
        require(_initialCollateral >= _minCollateral, "must start with min col");
        IERC20(_collateralAddress).transferFrom(msg.sender, address(pos), _initialCollateral);

        return address(pos);
    }
```

Same goes for **clonePosition** function. It first clones a position (deploys a new contract) and only then tries to transfer the **ZCHF** tokens.

```solidity
    function clonePosition(address position, uint256 _initialCollateral, uint256 _initialMint) public validPos(position) returns (address) {
        IPosition existing = IPosition(position);
        uint256 limit = existing.reduceLimitForClone(_initialMint);
        address pos = POSITION_FACTORY.clonePosition(position);
        zchf.registerPosition(pos);
        existing.collateral().transferFrom(msg.sender, address(pos), _initialCollateral);
        IPosition(pos).initializeClone(msg.sender, existing.price(), limit, _initialCollateral, _initialMint);
        return address(pos);
    }
```

# Floating pragma
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example an outdated compiler version that mught introduce bugs that affect the contract system negatively.

https://swcregistry.io/docs/SWC-103

### Instances
### ERC20.sol
```solidity
pragma solidity ^0.8.0;
```
### StablecoinBridge.sol
```solidity
pragma solidity ^0.8.0;
```
### ERC20PermitLight.sol
```solidity
pragma solidity ^0.8.0;
```
### Equity.sol
```solidity
pragma solidity >=0.8.0 <0.9.0;
```
### Frankencoin.sol
```solidity
pragma solidity ^0.8.0;
```
### IERC20.sol
```solidity
pragma solidity ^0.8.0;
```
### IERC677Receiver.sol
```solidity
pragma solidity ^0.8.0;
```
### IFrankencoin.sol
```solidity
pragma solidity ^0.8.0;
```
### IPosition.sol
```solidity
pragma solidity ^0.8.0;
```
### IReserve.sol
```solidity
pragma solidity ^0.8.0;
```
### MathUtil.sol
```solidity
pragma solidity >=0.8.0 <0.9.0;
```
### MintingHub.sol
```solidity
pragma solidity ^0.8.0;
```
### Ownable.sol
```solidity
pragma solidity ^0.8.0;
```
### Position.sol
```solidity
pragma solidity ^0.8.0;
```
### PositionFactory.sol
```solidity
pragma solidity ^0.8.0;
```
