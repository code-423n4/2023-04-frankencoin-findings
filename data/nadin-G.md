## Gas Optimizations
### Issue
## [G-01] Use nested if and, avoid multiple check combinations ( 04 instances )
## [G-02] Upgrade Solidity’s optimizer ( 01 instances )
## [G-03] Avoid using state variable in emit ( 05 instances )
## [G-04] Failure to check the zero address in the constructor causes the contract to be deployed again ( 03 instances )
## [G-05] Unchecked arithmetic ( 03 instances )
## [G-06] Use of named returns for local variables saves gas ( 04 instances )
## [G-07] >=/<= costs less gas than >/< ( 04 instances )
### Total: 28 instances  over 07  issues
## [G-01] Use nested if and, avoid multiple check combinations ( 04 instances )
### Description:
Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.
```
File: contracts/Position.sol
294:        if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L294)

```
File: contracts/Frankencoin.sol
84:      if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
85:      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
267:      if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();
```
[Frankencoin.sol#84-85](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L84-L85)
[Frankencoin#267](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L267)

## [G-02] Upgrade Solidity’s optimizer
#### Description:
Make sure Solidity’s optimizer is enabled. It reduces gas costs. If you want to gas optimize for contract deployment (costs less to deploy a contract) then set the Solidity optimizer at a low number. If you want to optimize for run-time gas costs (when functions are called on a contract) then set the optimizer to a high number.
Set the optimization value higher than 800 in your hardhat.config.js file.
```
96:          optimizer: {
97:              enabled: true,
98:              runs: 200
99:              },
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/hardhat.config.ts#L96-L99)
Please visit the following site for further information:
https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## [G-03] Avoid using state variable in emit ( 05 instances )
```
File: contracts/Position.sol
\\ @audit original 
69:        emit PositionOpened(_owner, original, _zchf, address(collateral), _liqPrice);
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L69)
```
File: contracts/MintingHub.sol
\\ @audit pos
146:        emit ChallengeStarted(msg.sender, address(position), _collateralAmount, pos);
177:        emit ChallengeStarted(copy.challenger, address(copy.position), copy.size, pos);
\\ @audit collateral
292:            emit PostPonedReturn(collateral, challenge.challenger, challenge.size);
```
[Link to code 146](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L146)
[Link to code 177](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L177)
[Link to code 292](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L292)

```
File: contracts/Equity.sol
\\ @audit proceeds
280:        emit Trade(msg.sender, -int(shares), proceeds, price());
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L280)

## [G-04] Failure to check the zero address in the constructor causes the contract to be deployed again ( 03 instances )
Zero address control is not performed in the constructor in all 3 contracts within the scope of the audit. Bypassing this check could cause the contract to be deployed by mistakenly entering a zero address. In this case, the contract will need to be redeployed. This means extra gas consumption as contract deployment costs are high.
```
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
[contracts/Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L50-L70)

```
    constructor(address _zchf, address factory) {
        zchf = IFrankencoin(_zchf);
        POSITION_FACTORY = IPositionFactory(factory);
    }
```
[contracts/MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L54-L57)

```
    constructor(address other, address zchfAddress, uint256 limit_){
        chf = IERC20(other);
        zchf = IFrankencoin(zchfAddress);
        horizon = block.timestamp + 52 weeks;
        limit = limit_;
    }
```
[contracts/StablecoinBridge.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L26-L31)

## [G-05] Unchecked arithmetic ( 03 instances )
The default “checked” behavior costs more gas when adding/diving/multiplying, because under-the-hood those checks are implemented as a series of opcodes that, prior to performing the actual arithmetic, check for under/overflow and revert if it is detected.
If it can statically be determined there is no possible way for your arithmetic to under/overflow (such as a condition in an if statement), surrounding the arithmetic in an unchecked block will save gas.
```
File: contracts/Position.sol
203:        uint256 horizon = block.timestamp + period;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L203

```
File: contracts/MintingHub.sol
217:            uint256 earliestEnd = block.timestamp + 30 minutes;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L217

```
File: contracts/Frankencoin.sol
88:      minters[_minter] = block.timestamp + _applicationPeriod;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L88

### Recommendation:
Place the arithmetic operations in an unchecked block

## [G-06] Use of named returns for local variables saves gas ( 04 instances )
You can have further advantages in terms of gas cost by simply using named return values as temporary local variable.

For instance, the code block below may be refactored as follows:
```
File: contracts/Position.sol
- 304:     function tryAvertChallenge(uint256 _collateralAmount, uint256 _bidAmountZCHF) external onlyHub returns (bool) {
+ 304:     function tryAvertChallenge(uint256 _collateralAmount, uint256 _bidAmountZCHF) external onlyHub returns (bool _status) {

             [ ... ]
- 315:            return false;
+ 315:           status_ = false;
316:        }
317:    }
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L304-L317

```
File: contracts/Equity.sol
225:    function canVoteFor(address delegate, address owner) internal view returns (bool) {
241:    function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool) {
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L225-L233
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L241-L255

```
File: contracts/StablecoinBridge.sol
75:    function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool){
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L75-L84

## [G-07] >=/<= costs less gas than >/< ( 04 instances )
The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas
```
File: contracts/Position.sol
349:        uint256 repayment = minted < volumeZCHF ? minted : volumeZCHF; // how much must be burned to make things even
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L349

```
File: contracts/Equity.sol
268:        uint256 newTotalShares = totalShares < 1000 * ONE_DEC18 ? 1000 * ONE_DEC18 : _mulD18(totalShares, _cubicRoot(_divD18(capitalBefore + investment, capitalBefore)));
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L268

```
File: contracts/Frankencoin.sol
238:      uint256 adjustedReservePPM = currentReserve < minterReserve_ ? reservePPM * currentReserve / minterReserve_ : reservePPM; // 18%
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L238

```
File: contracts/MathUtil.sol
26:            cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MathUtil.sol#L26