## Summary

### Gas Optimizations
| |Issue|Instances| |
|-|:-|:-:|:-:|
| [G&#x2011;01] | Avoid using state variable in emit | 1 |
| [G&#x2011;02] | Use nested if and, avoid multiple check combinations | 4 |
| [G&#x2011;03] | Zero value check in mint function, can save gas for minter  | 1 |

### [G&#x2011;01]  Avoid using state variable in emit
While emitting events, Passing parameter variables will cost less as compared to passing the storage values.
There are 1 instances of this issues.

```solidity
File: contracts/Position.sol

50    constructor(address _owner, address _hub, address _zchf, address _collateral, 
51        uint256 _minCollateral, uint256 _initialLimit, uint256 initPeriod, uint256 _duration,
52        uint256 _challengePeriod, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) {
53        require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values
54        setOwner(_owner);
55        original = address(this);
56        hub = _hub;
57        price = _liqPrice;
58        zchf = IFrankencoin(_zchf);
59        collateral = IERC20(_collateral);
60        mintingFeePPM = _mintingFeePPM;
61        reserveContribution = _reservePPM;
62        minimumCollateral = _minCollateral;
63        challengePeriod = _challengePeriod;
64        start = block.timestamp + initPeriod; // one week time to deny the position
65        cooldown = start;
66        expiration = start + _duration;
67        limit = _initialLimit;
68        
69        emit PositionOpened(_owner, original, _zchf, address(collateral), _liqPrice);
70    }
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L50-L70)

### Recommended Mitigation steps
By incorporating below recommendation, Some gas can be saved.

```solidity
File: contracts/Position.sol

    constructor(address _owner, address _hub, address _zchf, address _collateral, 
        uint256 _minCollateral, uint256 _initialLimit, uint256 initPeriod, uint256 _duration,
        uint256 _challengePeriod, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) {
        require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values
        setOwner(_owner);
        original = address(this);
        hub = _hub;
        price = _liqPrice;
        zchf = IFrankencoin(_zchf);
        collateral = IERC20(_collateral);
        mintingFeePPM = _mintingFeePPM;
        reserveContribution = _reservePPM;
        minimumCollateral = _minCollateral;
        challengePeriod = _challengePeriod;
        start = block.timestamp + initPeriod; // one week time to deny the position
        cooldown = start;
        expiration = start + _duration;
        limit = _initialLimit;
        
-        emit PositionOpened(_owner, original, _zchf, address(collateral), _liqPrice);
+        emit PositionOpened(_owner, original, _zchf, _collateral, _liqPrice);
    }
```

### [G&#x2011;02]  Use nested if and, avoid multiple check combinations
Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.
There are 4 instances of this issue.

```solidity
File: contracts/Position.sol

294        if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L294)

```solidity
File: contracts/Frankencoin.sol

84      if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
85      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L84-L85)

```solidity
File: contracts/Frankencoin.sol

267      if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L267)

### [G&#x2011;03]  Zero value check in mint function, can save gas for minter
mint() does not have zero value check which can waste lots of gas while called by minter for minting. It is recommended to have zero value check to prevent the wastage of gas.
There is 1 instance of this issue.

```solidity
File: contracts/Frankencoin.sol

165   function mint(address _target, uint256 _amount, uint32 _reservePPM, uint32 _feesPPM) override external 
      minterOnly {
166      uint256 usableMint = (_amount * (1000_000 - _feesPPM - _reservePPM)) / 1000_000; // rounding down is 
         fine
167      _mint(_target, usableMint);
168      _mint(address(reserve), _amount - usableMint); // rest goes to equity as reserves or as fees
169      minterReserveE6 += _amount * _reservePPM; // minter reserve must be kept accurately in order to ensure 
          we can get back to exactly 0
170   }
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L165-L170)

### Recommended Mitigation steps
Add zero value check in mint() function

```solidity
File: contracts/Frankencoin.sol

   function mint(address _target, uint256 _amount, uint32 _reservePPM, uint32 _feesPPM) override external 
      minterOnly {
+     require(_amount != 0, "invalid amount");
      uint256 usableMint = (_amount * (1000_000 - _feesPPM - _reservePPM)) / 1000_000; // rounding down is fine
      _mint(_target, usableMint);
      _mint(address(reserve), _amount - usableMint); // rest goes to equity as reserves or as fees
      minterReserveE6 += _amount * _reservePPM; // minter reserve must be kept accurately in order to ensure we can get back to exactly 0
   }
```
