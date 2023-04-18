## Low issue

## L-1 Add modifier in PositionFactory contract

if createNewPosition function Must be called through minting hub to be recognized as valid position you should add a modifier insted of let the responsability to the user, this can cause disorder in the protocol and can give some errors.

```
modifier onlyHub(){
 require(msg.sender == mintinHubAddress, " you are not minting hub contract")
}

function createNewPosition(
        address _owner,
        address _zchf,
        address _collateral,
        uint256 _minCollateral, //@audit if  Must be called through minting hub to be recognized as valid position. then create a modifier
        uint256 _initialLimit,
        uint256 initPeriod,
        uint256 _duration,
        uint256 _challengePeriod,
        uint32 _mintingFeePPM,
        uint256 _liqPrice,
        uint32 _reserve
    ) external onlyHub returns (address) {
```
