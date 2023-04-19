## Non-Critical Issues

| Number | Issues  | Instances |
| ------ | ------- | --------- |
| [N-01] | The `initPeriod` parameter should be `_initPeriod` | 1 |
| [N-02] | The `isMinter()` function should use the > operator instead of >= | 1 |


### [N-01] The `initPeriod` parameter should be `_initPeriod`

#### Line References

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L51
#### Description

The `initPeriod` parameter should be renamed to `_initPeriod` because it is a function parameter to clearly differentiate function parameters from state variables.

```solidity=50
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

    emit PositionOpened(_owner, original, _zchf, address(collateral), _liqPrice);
}
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L50-L70

#### Recommended Mitigation Steps
We suggest renaming the `initPeriod` parameter to `_initPeriod`.

### [N-02] The `isMinter()` function should use the `>` operator instead of `>=`

#### Line References

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L294
#### Description

The `minters[_minter]` value represents the exact timestamp when the minter's application period ends, then the current timestamp should be greater than the application period end timestamp for the minter to still be considered active. In this case, the `isMinter()` function should use the `>` operator instead of `>=`.

```solidity=293
function isMinter(address _minter) override public view returns (bool){
   return minters[_minter] != 0 && block.timestamp >= minters[_minter];
}
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L293-L295

#### Recommended Mitigation Steps
We suggest using `>` operator instead of `>=` for `block.timestamp >= minters[_minter]`.