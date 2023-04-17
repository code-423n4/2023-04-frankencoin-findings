# Gas Optimizations Report

Report Date: 2023-04-14 11:13:15

|        | Issue                      | Instances |
| ------ | :------------------------- | :-------: |
| [G-01] | Optimize names to save gas |    8    |

## [G-01] Optimize names to save gas

### Impact

public/external function names and public member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call.

### Findings

Total: 8

[contracts/Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/Equity.sol)

```solidity
    contract Equity is ERC20PermitLight, MathUtil, IReserve {
```
| Function Signature | Function Name |
|--------------------|---------------|
| 0xa035b1 | price() |
| 0xf97ed5 | canRedeem() |
| 0x10becd | canRedeem(address owner) |
| 0x9fecaa | votes(address holder) |
| 0x0d15fd | totalVotes() |
| 0x9aadfc | votes(address sender, address[] calldata helpers) |
| 0x40f187 | checkQualified(address sender, address[] calldata helpers) |
| 0x56f052 | delegateVoteTo(address delegate) |
| 0xf0dd1f | onTokenTransfer(address from, uint256 amount, bytes calldata) |
| 0xa35cd2 | calculateShares(uint256 investment) |
| 0x24e60d | redeem(address target, uint256 shares) |
| 0x7627c7 | calculateProceeds(uint256 shares) |
| 0xbce227 | restructureCapTable(address[] calldata helpers, address[] calldata addressesToWipe) |

[contracts/ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/ERC20.sol)

```solidity
    abstract contract ERC20 is IERC20 {
```
| Function Signature | Function Name |
|--------------------|---------------|
| 0x18160d | totalSupply() |
| 0xcc864d | balanceOf(address account) |
| 0x2ab3a7 | transfer(address recipient, uint256 amount) |
| 0x5f480c | allowance(address owner, address spender) |
| 0x19c54c | approve(address spender, uint256 value) |
| 0xb8847b | transferFrom(address sender, address recipient, uint256 amount) |
| 0x2a2b41 | transferAndCall(address recipient, uint256 amount, bytes calldata data) |

[contracts/ERC20PermitLight.sol](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/ERC20PermitLight.sol)

```solidity
    abstract contract ERC20PermitLight is ERC20 {
```

| Function Signature | Function Name |
|--------------------|---------------|
| 0x5b6d1a | permit(address owner,address spender,uint256 value,uint256 deadline,uint8 v,bytes32 r,bytes32 s) |
| 0x3644e5 | DOMAIN_SEPARATOR() |

[contracts/Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/Frankencoin.sol)

```solidity
    contract Frankencoin is ERC20PermitLight, IFrankencoin {
```

| Function Signature | Function Name |
|--------------------|---------------|
| 0xd38bb0 | minterReserve() |
| 0x91a0ac | equity() |
| 0xc23af5 | burn(uint256 _amount) |
| 0xb06ab2 | burn(uint256 amount, uint32 reservePPM) |
| 0xf4e7c7 | calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) |
| 0xf1ddc1 | burnFrom(address payer, uint256 targetTotalBurnAmount, uint32 _reservePPM) |
| 0xbed418 | calculateFreedAmount(uint256 amountExcludingReserve) |
| 0xab1498 | burnWithReserve(uint256 _amountExcludingReserve, uint32 _reservePPM) |

[contracts/MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/MintingHub.sol)

```solidity
    contract MintingHub {
```

| Function Signature | Function Name |
|--------------------|---------------|
| 0xc2331c | openPosition(address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,uint256 _mintingMaximum, uint256 _expirationSeconds, uint256 _challengeSeconds, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) |
| 0xbaafbe | openPosition(address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral, uint256 _mintingMaximum, uint256 _initPeriodSeconds, uint256 _expirationSeconds, uint256 _challengeSeconds, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) |
| 0x267b2a | clonePosition(address position, uint256 _initialCollateral, uint256 _initialMint) |
| 0x220c29 | launchChallenge(address _positionAddr, uint256 _collateralAmount) |
| 0x38e004 | splitChallenge(uint256 _challengeNumber, uint256 splitOffAmount) |
| 0xf6b2ed | minBid(uint256 challenge) |
| 0xf31ae6 | bid(uint256 _challengeNumber, uint256 _bidAmountZCHF, uint256 expectedSize) |
| 0x16da48 | end(uint256 _challengeNumber) |
| 0x647c90 | isChallengeOpen(uint256 _challengeNumber) |
| 0xb9c9e3 | end(uint256 _challengeNumber, bool postponeCollateralReturn) |
| 0x9f9ec2 | returnPostponedCollateral(address collateral, address target) |
| 0xab8384 | createNewPosition(address _owner,address _zchf, address _collateral, uint256 _minCollateral, uint256 _initialLimit,uint256 _initPeriodSeconds, uint256 _duration,uint256 _challengePeriod, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reserve) |
| 0x0c5961 | clonePosition(address _existing) |

[contracts/Position.sol](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/Position.sol)

```solidity
    contract Position is Ownable, IPosition, MathUtil {
```

| Function Signature | Function Name |
|--------------------|---------------|
| 0x5407ea | initializeClone(address owner, uint256 _price, uint256 _limit, uint256 _coll, uint256 _mint) |
| 0xf3a887 | reduceLimitForClone(uint256 _minimum) |
| 0xe129b6 | deny(address[] calldata helpers, string calldata message) |
| 0x9d9467 | getUsableMint(uint256 totalMint, bool afterFees) |
| 0x3f4e37 | adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) |
| 0xb72a12 | adjustPrice(uint256 newPrice) |
| 0x60ccae | mint(address target, uint256 amount) |
| 0x383ef4 | calculateCurrentFee() |
| 0x34c1eb | repay(uint256 amount) |
| 0x33b2a5 | withdraw(address token, address target, uint256 amount) |
| 0x3e7c16 | withdrawCollateral(address target, uint256 amount) |
| 0x918043 | notifyChallengeStarted(uint256 size) |
| 0x7a6512 | tryAvertChallenge(uint256 _collateralAmount, uint256 _bidAmountZCHF) |
| 0x7fa9bf | notifyChallengeSucceeded(address _bidder, uint256 _bid, uint256 _size) |
| 0xc2b6b5 | isClosed() |

[contracts/PositionFactory.sol](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/PositionFactory.sol)

```solidity
    contract PositionFactory{
```

| Function Signature | Function Name |
|--------------------|---------------|
| 0x3c3626 | createNewPosition(address _owner, address _zchf,address _collateral, uint256 _minCollateral,uint256 _initialLimit, uint256 initPeriod, uint256 _duration, uint256 _challengePeriod, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reserve) |
| 0x0c5961 | clonePosition(address _existing) |

[contracts/StablecoinBridge.sol](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/StablecoinBridge.sol)

```solidity
    contract StablecoinBridge {
```

| Function Signature | Function Name |
|--------------------|---------------|
| 0xe0b7c8 | mint(uint256 amount) |
| 0x60ccae | mint(address target, uint256 amount) |
| 0x27199c | burn(uint256 amount) |
| 0x48539d | burn(address target, uint256 amount) |
| 0xf0dd1f | onTokenTransfer(address from, uint256 amount, bytes calldata) |
