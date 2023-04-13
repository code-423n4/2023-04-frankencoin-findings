## `_initPeriodSeconds` can be declare as constant
 
In code https://github.com/code-423n4/2023-04-frankencoin/blob/f86279e76fd9f810d2a25243012e1be4191a547e/contracts/MintingHub.sol#L59-L65
```solidity
    function openPosition(
        address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
        uint256 _mintingMaximum, uint256 _expirationSeconds, uint256 _challengeSeconds,
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address) {
            return openPosition(_collateralAddress, _minCollateral, _initialCollateral, _mintingMaximum,
            7 days, _expirationSeconds, _challengeSeconds, _mintingFeePPM, _liqPrice, _reservePPM);
    }
```
_initPeriodSeconds is hardcoded to 7days. Recommend declare it as constant to save gas in `openPosition` function and deploy MintingHub contract.

```solidity
uint256 public constant initPeriodSeconds = 7 days;
```