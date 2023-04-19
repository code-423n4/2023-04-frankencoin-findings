# Open Position Minting Time Limit Misleading

## Description

The `MintingHub` smart contract features a open position system that allows any user to open a position to mint `Frankencoins`. Every user can open a new position with calling `openPosition` function. However according to the documentation of frankencoin [when opening a position](https://docs.frankencoin.com/positions/open), it is already provided with some collateral, but nothing is minted yet. Minting is not possible until the initialization period of `seven` days has passed.

If we take a look to the `openPosition` function it can be seen that this `seven day` restriction can be bypassed.


```javascript
    function openPosition(
        address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
        uint256 _mintingMaximum, uint256 _expirationSeconds, uint256 _challengeSeconds,
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address) {
            return openPosition(_collateralAddress, _minCollateral, _initialCollateral, _mintingMaximum,
            7 days, _expirationSeconds, _challengeSeconds, _mintingFeePPM, _liqPrice, _reservePPM);
    }
```

Given `openPosition` function is setting parameter correctly, however it is calling another `openPosition` function with setting `7 days` value to the `_initPeriodSeconds` parameter. 

```javascript
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
                _initialCollateral,
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
        zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE);
        IERC20(_collateralAddress).transferFrom(msg.sender, address(pos), _initialCollateral);

        return address(pos);
    }
```
However it can be seen that this second `openPosition` function is public and accessible for anyone. If we directly call this function instead of the first `openPosition` it is possible to bypass `seven days` restriction. This function is calling `createNewPosition` function function from `PositionFactory.sol` contract. 
```javascript
    function createNewPosition(address _owner, address _zchf, address _collateral, 
        uint256 _minCollateral, uint256 _initialCollateral, 
        uint256 _initialLimit, uint256 initPeriod, uint256 _duration, uint256 _challengePeriod, 
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reserve) 
        external returns (address) {
        return address(new Position(_owner, msg.sender, _zchf, _collateral, 
            _minCollateral, _initialCollateral, _initialLimit, initPeriod, _duration, 
            _challengePeriod, _mintingFeePPM, _liqPrice, _reserve));
    }
```
And it is calling `new Position()`. So if we take a look to the `Position` contract 
```javascript
  constructor(address _owner, address _hub, address _zchf, address _collateral, 
        uint256 _minCollateral, uint256 _initialCollateral, 
        uint256 _initialLimit, uint256 initPeriod, uint256 _duration, uint256 _challengePeriod, uint32 _mintingFeePPM, 
        uint256 _liqPrice, uint32 _reservePPM) {
        require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values
        setOwner(_owner);
        original = address(this);
        hub = _hub;
        price = _liqPrice;
        zchf = IFrankencoin(_zchf);
        collateral = IERC20(_collateral);
        mintingFeePPM = _mintingFeePPM;
        reserveContribution = _reservePPM;
        if(_initialCollateral < _minCollateral) revert InsufficientCollateral();
        minimumCollateral = _minCollateral;
        challengePeriod = _challengePeriod;
        start = block.timestamp + initPeriod; // one week time to deny the position
        cooldown = start;
        expiration = start + _duration;
        limit = _initialLimit;
        
        emit PositionOpened(_owner, original, _zchf, address(collateral), _liqPrice);
    }
```
It is setting given `initPeriod` parameter to the `start` variable. And it is only checking if given `initPeriod` is higher than or equal to the `3 days` instead of `7 days`. So with the help of second function any user can open a position with 3 day restriction and it bypasses the restriction limit defined in the first place. 

## POC
* User calls second `openPosition` function with giving `3 days` for variable `_initPeriodSeconds` and it will open a position.
* No one would be able to deny that position after 3 days.

## Recommendation

It is recommended that the visibility of the second `openPosition` function be changed from `public` to `internal`. Here is the updated code for the `openPosition` function:


```javascript
function openPosition(
        address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
        uint256 _mintingMaximum, uint256 _initPeriodSeconds, uint256 _expirationSeconds, uint256 _challengeSeconds,
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) internal returns (address) {
        IPosition pos = IPosition(
            POSITION_FACTORY.createNewPosition(
                msg.sender,
                address(zchf),
                _collateralAddress,
                _minCollateral,
                _initialCollateral,
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
        zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE);
        IERC20(_collateralAddress).transferFrom(msg.sender, address(pos), _initialCollateral);

        return address(pos);
    }
```