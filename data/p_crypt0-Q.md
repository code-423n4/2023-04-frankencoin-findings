# Position.sol

## [Informational] Constructor should use variable for initPeriod, rather than magic number.

Currently, the `Position.sol` constructor validates the `initPeriod` to be greater than or equal to 3 days, however, recommends a higher value.

```solidity
constructor(address _owner, address _hub, address _zchf, address _collateral, 
        uint256 _minCollateral, uint256 _initialLimit, uint256 initPeriod, uint256 _duration,
        uint256 _challengePeriod, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) {
        require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values //@audit this ought to be either a constant, or variable period rather than magic number.
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
        
        emit PositionOpened(_owner, original, _zchf, address(collateral), _liqPrice);
    }
```

The issues with using a magic number here, is that if `initPeriod` requires a different value in the future - say, at least 4 days becomes the normal `initPeriod` - then the codebase itself has to change rather than a variable update.

### Recommendation
Two paths:

First, use a `constant` variable called `minInitPeriod` and set it equal to the desired number of days for a minimum `initPeriod`


Alternatively, use a `uint` to create a `private` changeable variable for a `minInitPeriod`:
	2 a) create a `setMinInitPeriod(uint newMinInitPeriod)` method which is access restricted to the owner of the contract.
	2 b) use the `minInitPeriod` variable in checks.

### Update from hermel#2851 via discord

Hermel described the finding as a good spot and that 3 days is currently the desired amount of time for a minInitPeriod. The comment was described as being outdated, however, if the initPeriod has changed in the past, this is precedent for it perhaps changing again in the future so my recommendations stand as informational.