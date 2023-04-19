# QA Report for Frankencoin contest
## Overview
During the audit, 3 low and 7 non-critical issues were found.

№ | Title | Risk Rating  | Instance Count
--- | --- | --- | ---
L-1 | Function transferAndCall is not safe | Low | 1
L-2 | Use SafeCast Library | Low | 1
L-3 | Check that amount to mint > 0 | Low | 5
NC-1 | Order of Layout | Non-Critical | 7
NC-2 | Inconsistency when using the number 1000_000 | Non-Critical | 7
NC-3 | Inconsistency when using uint and uint256 | Non-Critical | 3
NC-4 | Prevent zero transfers | Non-Critical | 2
NC-5 | Natspec is incomplete | Non-Critical | 1
NC-6 | No space between the control structures | Non-Critical | 1
NC-7 | Missing leading underscore | Non-Critical | 33

## Low Risk Findings(3)
### L-1. Function transferAndCall is not safe
##### Description
The function ```transferAndCall``` is vulnerable to reentrancy.
##### Instances
- [```function transferAndCall(address recipient, uint256 amount, bytes calldata data) external override returns (bool) {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L162)

##### Recommendation
Avoid using it.
#
### L-2. Use SafeCast Library
##### Description
Downcasting from uint256/int256 in Solidity does not revert on overflow. This can easily result in undesired exploitation or bugs, since developers usually assume that overflows raise errors. SafeCast restores this intuition by reverting the transaction when such an operation overflows.
##### Instances
- [```totalVotesAtAnchor = uint192(totalVotes() - roundingLoss - lostVotes);```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L146)

##### Recommendation
It is better to use [safe casting library](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast).
#
### L-3. Check that amount to mint > 0
##### Instances
- [```_mint(_target, usableMint);```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L167) 
- [```_mint(address(reserve), _amount - usableMint); // rest goes to equity as reserves or as fees```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L168) 
- [```_mint(_target, _amount);```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L173) 
- [```_mint(msg.sender, _amount - reserveLeft);```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L286) 
- [```mint(msg.sender, amount);```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L37) 

##### Recommendation
For example, add the check:
```
if (usableMint == 0) revert ZeroAmount();
_mint(_target, usableMint);
```
#
## Non-Critical Risk Findings(7)
### NC-1. Order of Layout
##### Description
According to [Order of Layout](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout), inside each contract, library or interface, use the following order:
1) Type declarations
2) State variables
3) Events
4) Modifiers
5) Functions
##### Instances
- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L366
- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L373
- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L380
- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L387
- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L115
- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L266
- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L49

##### Recommendation
Place modifiers before constructor.
#
### NC-2. Inconsistency when using the number 1000_000
##### Description
In some cases, 1000000 is used, and in some - 1000_000.
##### Instances
1000000:
- [```return minterReserveE6 / 1000000;```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L118) 
- [```uint256 theoreticalReserve = _reservePPM * mintedAmount / 1000000;```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L205) 
- [```return 1000000 * amountExcludingReserve / (1000000 - adjustedReservePPM); // 41 / (1-18%) = 50```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L239) 

1000_000:
- [```return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L122) 
- [```return totalMint * (1000_000 - reserveContribution) / 1000_000;```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L124) 
- [```uint256 reward = (volume * CHALLENGER_REWARD) / 1000_000;```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L265) 
- [```uint256 usableMint = (_amount * (1000_000 - _feesPPM - _reservePPM)) / 1000_000; // rounding down is fine```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L166) 

##### Recommendation
Stick to one style.
#
### NC-3. Inconsistency when using uint and uint256
##### Description
Some variables is declared as ```uint``` and some as ```uint256```.
##### Instances
There are 4 cases with uint when the rest are with uint256:
- [```event Trade(address who, int amount, uint totPrice, uint newprice);```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L91) 
- [```for (uint i=0; i<helpers.length; i++){```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L192) 
- [```for (uint j=i+1; j<helpers.length; j++){```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L196) 

##### Recommendation
Stick to one style.
#
### NC-4. Prevent zero transfers
##### Description
Check that amount to transfer > 0.
##### Instances
- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L142
- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L279

#
### NC-5. Natspec is incomplete
##### Instances
Parameter ```roundingLoss``` is missing.
```
* @notice Decrease the total votes anchor when tokens lose their voting power due to being moved
* @param from      sender
* @param amount    amount to be sent
*/
```
- [```function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L144)

##### Recommendation
Add the description for the ```roundingLoss```.
#
### NC-6. No space between the control structures
##### Description
According to [Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#control-structures), there should be a single space between the control structures ```if```, ```while```, and ```for``` and the parenthetic block representing the conditional.
##### Instances
- [```if(_coll < minimumCollateral) revert InsufficientCollateral();```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L77) 

##### Recommendation
Change:
```
if(...) {
    ...
}
```
to:
```
if (...) {
    ...
}
```
#
### NC-7. Missing leading underscores
##### Description
Internal and private constants, immutables and functions should have a leading underscore.
##### Instances
- [```function collateralBalance() internal view returns (uint256){```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L169) 
- [```function mintInternal(address target, uint256 amount, uint256 collateral_) internal {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L193) 
- [```function restrictMinting(uint256 period) internal {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L202) 
- [```function repayInternal(uint256 burnable) internal {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L232) 
- [```function notifyRepaidInternal(uint256 amount) internal {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L240) 
- [```function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L268) 
- [```function checkCollateral(uint256 collateralReserve, uint256 atPrice) internal view {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L282) 
- [```function emitUpdate() internal {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L286) 
- [```IPositionFactory private immutable POSITION_FACTORY; // position contract to clone```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L28) 
- [```function minBid(Challenge storage challenge) internal view returns (uint256) {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L188) 
- [```function returnCollateral(Challenge storage challenge, bool postpone) internal {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L287) 
- [```uint256 private constant MINIMUM_EQUITY = 1000 * ONE_DEC18;```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L41) 
- [```uint32 private constant QUORUM = 300;```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L46) 
- [```uint8 private constant BLOCK_TIME_RESOLUTION_BITS = 24;```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L51) 
- [```uint192 private totalVotesAtAnchor;  // Total number of votes at the anchor time, see comment on the um```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L75) 
- [```uint64 private totalVotesAnchorTime; // 40 Bit for the block number, 24 Bit sub-block time resolution```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L76) 
- [```mapping (address => uint64) private voteAnchor; // 40 Bit for the block number, 24 Bit sub-block time resolution```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L88) 
- [```function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L144) 
- [```function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L157) 
- [```function anchorTime() internal view returns (uint64){```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L172) 
- [```function canVoteFor(address delegate, address owner) internal view returns (bool) {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L225) 
- [```function calculateSharesInternal(uint256 capitalBefore, uint256 investment) internal view returns (uint256) {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L266) 
- [```uint256 private minterReserveE6;```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L39) 
- [```function allowanceInternal(address owner, address spender) internal view override returns (uint256) {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L102) 
- [```uint256 internal constant INFINITY = (1 << 255);```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L47) 
- [```function allowanceInternal(address owner, address spender) internal view virtual returns (uint256) {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L97) 
- [```function mintInternal(address target, uint256 amount) internal {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L49) 
- [```function burnInternal(address zchfHolder, address target, uint256 amount) internal {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L67) 
- [```function createClone(address target) internal returns (address result) {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L37) 
- [```uint256 internal constant ONE_DEC18 = 10**18;```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L10) 
- [```uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L11) 
- [```function setOwner(address newOwner) internal {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L39) 
- [```function requireOwner(address sender) internal view {```](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L45) 

##### Recommendation
Add leading underscores where needed.