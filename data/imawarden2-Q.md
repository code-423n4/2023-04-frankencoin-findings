## [Q-1] Failed ERC20 collateral transfers may adversely impact protocol functioning

Slither produces a number of error listings related to unchecked ERC20 transfer return values. These can safely be ignored when the transfers are from Frankencoin protocol tokens (i.e. ZCHF) with known behavior, but not necessarily for other external tokens. Token implementations vary with respect to their return values. Ignoring the return values may result in a failed token transfer being treated as successful, which may have various downstream impacts resulting from inaccurate balance data.

```
MintingHub.openPosition(address,uint256,uint256,uint256,uint256,uint256,uint256,uint32,uint256,uint32) (contracts/MintingHub.sol#88-113) ignores return value by IERC20(_collateralAddress).transferFrom(msg.sender,address(pos),_initialCollateral) (contra
cts/MintingHub.sol#110)

MintingHub.clonePosition(address,uint256,uint256) (contracts/MintingHub.sol#124-132) ignores return value by existing.collateral().transferFrom(msg.sender,address(pos),_initialCollateral) (contracts/MintingHub.sol#129)

MintingHub.launchChallenge(address,uint256) (contracts/MintingHub.sol#140-148) ignores return value by IERC20(position.collateral()).transferFrom(msg.sender,address(this),_collateralAmount) (contracts/MintingHub.sol#142)

MintingHub.bid(uint256,uint256,uint256) (contracts/MintingHub.sol#199-229) ignores return value by challenge.position.collateral().transfer(msg.sender,challenge.size) (contracts/MintingHub.sol#211)

MintingHub.returnPostponedCollateral(address,address) (contracts/MintingHub.sol#281-285) ignores return value by IERC20(collateral).transfer(target,amount) (contracts/MintingHub.sol#284)

MintingHub.returnCollateral(MintingHub.Challenge,bool) (contracts/MintingHub.sol#287-296) ignores return value by challenge.position.collateral().transfer(challenge.challenger,challenge.size) (contracts/MintingHub.sol#294)

Position.adjust(uint256,uint256,uint256) (contracts/Position.sol#132-152) ignores return value by collateral.transferFrom(msg.sender,address(this),newCollateral - colbal) (contracts/Position.sol#138)

Position.internalWithdrawCollateral(address,uint256) (contracts/Position.sol#268-276) ignores return value by IERC20(collateral).transfer(target,amount) (contracts/Position.sol#269)
```

### Remediations to Consider

Consider implementing transfers of external tokens to include safety checks, for example OpenZeppelin's [SafeERC20 library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol).