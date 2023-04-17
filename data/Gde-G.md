## Require could be called earlier to save gas

Always try reverting transactions as early as possible when using require statements. In case a transaction revert occurs, the user will pay the gas up until the revert was executed (not afterwards).

L109 `require(_initialCollateral >= _minCollateral, "must start with min col");` could be called first, before creating the position:

```solidity
File: contracts/MintingHub.sol
088:     function openPosition(
089:         address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
090:         uint256 _mintingMaximum, uint256 _initPeriodSeconds, uint256 _expirationSeconds, uint256 _challengeSeconds,
091:         uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address) {
092:         IPosition pos = IPosition(
093:             POSITION_FACTORY.createNewPosition(
094:                 msg.sender,
095:                 address(zchf),
096:                 _collateralAddress,
097:                 _minCollateral,
098:                 _mintingMaximum,
099:                 _initPeriodSeconds,
100:                 _expirationSeconds,
101:                 _challengeSeconds,
102:                 _mintingFeePPM,
103:                 _liqPrice,
104:                 _reservePPM
105:             )
106:         );
107:         zchf.registerPosition(address(pos));
108:         zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE);
109:         require(_initialCollateral >= _minCollateral, "must start with min col");
```