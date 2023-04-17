## [GAS-01] `> 0` is less efficient than `!= 0` and `!= 0` costs less gas compared to `> 0` for unsigned integers.
There are 6 instances of this issue:

[contracts/Position.sol#L381](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L381)
```solidity
    modifier noChallenge() {
        if (challengedAmount > 0) revert Challenged();
        _;
    }
```
[contracts/MintingHub.sol#L203](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L203)
```solidity
if (challenge.bid > 0) {
            zchf.transfer(challenge.bidder, challenge.bid); // return old bid
        }
```
[contracts/Equity.sol#L114](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L114)
```solidity
 if (amount > 0){...}
```
[contracts/Frankencoin.sol#L84](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84)
```solidity
if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
```
[contracts/Frankencoin.sol#L85](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L85)
```solidity
if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
```
[contracts/Frankencoin.sol#L104](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L104)
```solidity
if (explicit > 0){...}
```
Replace `> 0` with `!= 0`

## [GAS-02] No need to read `limit` second time. Save gas
[contracts/MintingHub.sol#L126](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L126)
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
Change [line 130](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L130) it to `IPosition(pos).initializeClone(msg.sender, existing.price(), existing.reduceLimitForClone(_initialMint), _initialCollateral, _initialMint);` and delete `uint256 limit = existing.reduceLimitForClone(_initialMint);` and gas is saved this way.
