**Low-Risk Issues List**

|  | Issue | Instances |
| --- | --- | --- |
| [L-01] | Use safeTransfer() / safeTransferFrom() instead of transfer() / transferFrom() for ERC20 | 6 |
| [L-02] | Signature Malleability of EVM's ecrecover() | 1 |
| [L-03] | Unsafe casting from uint256 | 4 |
| [L-04] | Owner can renounce Ownership | 6 |
| [L-05] | Single-step transfer ownership | 2 |
| [L-06] | In the constructor there is no return of incorrect address identification | 2 |
| [L-07] | Project Upgrade and Stop Scenario should be | 1 |
| [L-08] | Add to blacklist function | 1 |
| [L-09] | Function Calls in Loop Could Lead to Denial of Service due to Array length not being checked | 2 |
| [L-10] | Use uint256 instead uint | 4 |

Total 10 issues

**Non-Critical Issues List**

|  | Issue | Instances |
| --- | --- | --- |
| [Q-01] | require()/revert() statements without descriptive reason strings | 15 |
| [Q-02] | Event is missing indexed fields | 13 |
| [Q-03] | Use scientific notation(1e18) instead of exponentiation(10**18) | 3 |
| [Q-04] | Non-constant/immutable variables with all capital letters | 17 |
| [Q-05] | NatSpec comments should be in a common format | All Contracts |
| [Q-06] | For modern and more readable code; update import usage | All contracts |
| [Q-07] | Assembly Codes Specific – Should Have Comments | 1 |
| [Q-08] | Take advantage of Custom Error's return value property | All Errors |
| [Q-09] | Function Naming suggestions | 22 |
| [Q-10] | Order of functions not following the Style Guide | 7 |
| [Q-11] | Long lines are not suitable for the Solidity Style Guide | 56 |
| [Q-12] | Floating pragma | All contracts |

Total 12 issues

## [L-01] Use safeTransfer()/safeTransferFrom() instead of transfer()/transferFrom() for ERC20

Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens.
For example, Tether (USDT)'s transfer() and transferFrom() functions do not return booleans as the specification requires,
and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and
therefore the calls made revert.

Use OpenZeppelin's SafeERC20's safeTransfer() and safeTransferFrom() instead.

```solidity
File: .\contracts\MintingHub.sol

110: IERC20(_collateralAddress).transferFrom(msg.sender, address(pos), _initialCollateral);
263: IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);
284: IERC20(collateral).transfer(target, amount);

204: zchf.transfer(challenge.bidder, challenge.bid); // return old bid
268: zchf.transfer(owner, effectiveBid - fundsNeeded);
272: zchf.transfer(challenge.challenger, reward); // pay out the challenger reward

```

```solidity
File: .\contracts\Position.sol

228: IERC20(zchf).transferFrom(msg.sender, address(this), amount);
253: IERC20(token).transfer(target, amount);
269: IERC20(collateral).transfer(target, amount);
```

```solidity
File: .\contracts\Equity.sol

279: zchf.transfer(target, proceeds);

```

```solidity
File: .\contracts\StablecoinBridge.sol

69: chf.transfer(target, amount);

```

## [L-02] Signature Malleability of EVM's ecrecover()

**Description:** Description: The function calls the Solidity ecrecover() function directly to verify the given signatures. However, the ecrecover() EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

Although a replay attack seems not possible for this contract, I recommend using the battle-tested OpenZeppelin's ECDSA library.

*Recommendation:** Use the ecrecover function from OpenZeppelin's ECDSA library for signature verification. (Ensure using a version > 4.7.3 for there was a critical bug >= 4.1.0 < 4.7.3).

```solidity
File: contracts\ERC20PermitLight.sol

32: unchecked { // unchecked to save a little gas with the nonce increment...
33:        address recoveredAddress = ecrecover(
34:            keccak256(
```

## [L-03] Unsafe casting from `uint256`

Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256.

```solidity
File: .\\contracts\\Equity.sol

146: totalVotesAtAnchor = uint192(totalVotes() - roundingLoss - lostVotes);
161: voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); 
173: return uint64(block.number << BLOCK_TIME_RESOLUTION_BITS);

```

```solidity
File: .\\contracts\\Position.sol

187: return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));

```

## [L-04] Owner can renounce Ownership

The contract uses OpenZeppelin's Ownable(Upgradable).sol and contains onlyOwner functions. Consider implementing custom logic for renounceOwnership() to handle renouncing ownership.

`onlyOwner` functions:

```solidity
File: .\\contracts\\Position.sol

132: function adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) public onlyOwner {
159: function adjustPrice(uint256 newPrice) public onlyOwner noChallenge {
177: function mint(address target, uint256 amount) public onlyOwner noChallenge noCooldown alive {
227: function repay(uint256 amount) public onlyOwner {
249: function withdraw(address token, address target, uint256 amount) external onlyOwner {
263: function withdrawCollateral(address target, uint256 amount) public onlyOwner noChallenge noCooldown {

```

## [L-05] Single-step transfer ownership

Consider using transferOwnership() from Ownable2Step.sol (Ownable2StepUpgradeable.sol) for a more secure 2-step ownership transfer.

```solidity
File: .\\contracts\\MintingHub.sol

7: import "./Ownable.sol";
```

```solidity
File: .\\contracts\\Position.sol

8: import "./Ownable.sol";
```

## [L-06] In the `constructor` there is no return of incorrect address identification

In case of incorrect address definition in the `constructor` , there is no way to fix it because of the variables are immutable

It is recommended to fix the architecture:

- Address definitions can be done by changeable architecture 

- Because of `owner = address(0)` at the end of the constructor, there is no way to fix it, so the owner's authority can be maintained.

```solidity
File: contracts\Equity.sol

constructor(Frankencoin zchf_) ERC20(18) {
    zchf = zchf_;
}
```

```solidity
File: contracts\Frankencoin.sol

constructor(uint256 _minApplicationPeriod) ERC20(18){
  MIN_APPLICATION_PERIOD = _minApplicationPeriod;
}
```

## [L-07] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

[https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol](https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol)

## [L-08] Add to *blacklist* function

**Description:** Cryptocurrency mixing service, Tornado Cash, has been blacklisted in the OFAC. A lot of blockchain companies, token projects, NFT Projects have `blacklisted` all Ethereum addresses owned by Tornado Cash listed in the US Treasury Department's sanction against the protocol. [https://home.treasury.gov/policy-issues/financial-sanctions/recent-actions/20220808](https://home.treasury.gov/policy-issues/financial-sanctions/recent-actions/20220808) In addition, these platforms even ban accounts that have received ETH on their account with Tornadocash.

Some of these Projects; USDC ([https://www.circle.com/en/usdc](https://www.circle.com/en/usdc)) Flashbots ([https://www.paradigm.xyz/portfolio/flashbots](https://www.paradigm.xyz/portfolio/flashbots) ) Aave ([https://aave.com/](https://aave.com/)) Uniswap Balancer Infura Alchemy Opensea dYdX

Details on the subject; [https://twitter.com/bantg/status/1556712790894706688?s=20&t=HUTDTeLikUr6Dv9JdMF7AA](https://twitter.com/bantg/status/1556712790894706688?s=20&t=HUTDTeLikUr6Dv9JdMF7AA)

`The ban on Tornado Cash makes little sense, because in the end, no one can prevent people from using other mixer smart contracts, or forking the existing ones. It neither hinders cybercrime, nor privacy.`

Here is the most beautiful and close to the project example; Manifold

Manifold Contract [https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code](https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code)

```solidity
modifier nonBlacklistRequired(address extension) {
   require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
   _;
}
```

Recommended Mitigation Steps: add to Blacklist function and modifier.

## [L-09] **Function Calls in Loop Could Lead to DoS due to Array length not being checked**

Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to gas limitations or failed transactions. Consider bounding the loop length if the array is expected to be growing and/or handling a huge list of elements to avoid unnecessary gas wastage and denial of service.

```solidity
File: contracts\Equity.sol

function restructureCapTable(address[] calldata helpers, address[] calldata addressesToWipe) public {
    require(zchf.equity() < MINIMUM_EQUITY);
    checkQualified(msg.sender, helpers);
    for (uint256 i = 0; i<addressesToWipe.length; i++){
        address current = addressesToWipe[0];
        _burn(current, balanceOf(current));
    }
}

function votes(address sender, address[] calldata helpers) public view returns (uint256) {
    uint256 _votes = votes(sender);
    for (uint i=0; i<helpers.length; i++){
        address current = helpers[i];
        require(current != sender);
        require(canVoteFor(sender, current));
        for (uint j=i+1; j<helpers.length; j++){
            require(current != helpers[j]); // ensure helper unique
        }
        _votes += votes(current);
    }
    return _votes;
}

```

## [L-10] Use `uint256` instead `uint`

Some developers prefer to use `uint256` because it is consistent with other uint data types, which also specify their size, and also because making the size of the data explicit reminds the developer and the reader how much data they've got to play with, which may help prevent or detect bugs.

```solidity
File: contracts\Equity.sol

for (uint i=0; i<helpers.length; i++){
for (uint j=i+1; j<helpers.length; j++){
event Trade(address who, int amount, uint totPrice, uint newprice); // amount pos or neg for mint or redemption
```

## [Q-01] require()/revert() statements without descriptive reason strings

Consider adding descriptive reason strings to require() and revert() statements for better error handling and debugging.

```solidity
File: .\\contracts\\Equity.sol

194: require(current != sender);
195: require(canVoteFor(sender, current));
197: require(current != helpers[j]); // ensure helper unique
276: require(canRedeem(msg.sender));
310: require(zchf.equity() < MINIMUM_EQUITY);

```

```solidity
File: .\\contracts\\ERC20.sol

152: require(recipient != address(0));
180: require(recipient != address(0));

```

```solidity
File: .\\contracts\\MintingHub.sol

116: require(zchf.isPosition(position) == address(this), "not our pos");
158: require(challenge.challenger != address(0x0));
171: require(challenge.size >= min);
172: require(copy.size >= min);
254: require(challenge.challenger != address(0x0));

```

```solidity
File: .\\contracts\\Position.sol

53: require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values

```

```solidity
File: .\\contracts\\StablecoinBridge.sol

50: require(block.timestamp <= horizon, "expired");
51: require(chf.balanceOf(address(this)) <= limit, "limit");

```

## [Q-02] Event is missing indexed fields

Index event fields to make them more quickly accessible to off-chain tools that parse events. However, be aware that each indexed field costs extra gas during emission.

```solidity
File: .\\contracts\\Equity.sol

91: event Trade(address who, int amount, uint totPrice, uint newprice); // amount pos or neg for mint or redemption

```

```solidity
File: .\\contracts\\Frankencoin.sol

52: event MinterApplied(address indexed minter, uint256 applicationPeriod, uint256 applicationFee, string message);
53: event MinterDenied(address indexed minter, string message);

```

```solidity
File: .\\contracts\\IERC20.sol

86: event Transfer(address indexed from, address indexed to, uint256 value);
92: event Approval(address indexed owner, address indexed spender, uint256 value);

```

```solidity
File: .\\contracts\\MintingHub.sol

48: event ChallengeStarted(address indexed challenger, address indexed position, uint256 size, uint256 number);
49: event ChallengeAverted(address indexed position, uint256 number);
50: event ChallengeSucceeded(address indexed position, uint256 bid, uint256 number);
51: event NewBid(uint256 challengedId, uint256 bidAmount, address bidder);
52: event PostPonedReturn(address collateral, address indexed beneficiary, uint256 amount);

```

```solidity
File: .\\contracts\\Position.sol

41: event PositionOpened(address indexed owner, address original, address zchf, address collateral, uint256 price);
42: event MintingUpdate(uint256 collateral, uint256 price, uint256 minted, uint256 limit);
43: event PositionDenied(address indexed sender, string message); // emitted if closed by governance

```

## [Q-03] Use scientific notation(1e18) instead of exponentiation(10**18)

Consider using scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10\*\*18) for better coding practice and readability.

```solidity
File: .\\contracts\\Frankencoin.sol

25: uint256 public constant MIN_FEE = 1000 * (10**18);

```

```solidity
File: .\\contracts\\MathUtil.sol

10: uint256 internal constant ONE_DEC18 = 10**18;

```

```solidity
File: .\\contracts\\MintingHub.sol

20: uint256 public constant OPENING_FEE = 1000 * 10**18;

```

## [Q-04] Non-constant/immutable variables with all capital letters

Variable names that consist of all capital letters should be reserved for constant/immutable variables.

```solidity
File: .\\contracts\\Equity.sol

61: Frankencoin immutable public zchf;

```

```
File: .\\contracts\\Frankencoin.sol

31: IReserve override public immutable reserve;

```

```solidity
File: .\\contracts\\MintingHub.sol

30: IFrankencoin public immutable zchf; // currency

```

```solidity
File: .\\contracts\\Position.sol

24: uint256 public immutable challengePeriod; // challenge period in seconds
29: uint256 public immutable start; // timestamp when minting can start
30: uint256 public immutable expiration; // timestamp at which the position expires
32: address public immutable original; // originals point to themselves, clone to their origin
33: address public immutable hub; // the hub this position was created by
34: IFrankencoin public immutable zchf; // currency
35: IERC20 public override immutable collateral; // collateral
36: uint256 public override immutable minimumCollateral; // prevent dust amounts
38: uint32 public immutable mintingFeePPM;
39: uint32 public immutable reserveContribution; // in ppm

```

```solidity
File: .\\contracts\\StablecoinBridge.sol

13: IERC20 public immutable chf; // the source stablecoin
14: IFrankencoin public immutable zchf; // the Frankencoin
19: uint256 public immutable horizon;
24: uint256 public immutable limit;

```

## [Q-05] NatSpec comments should be in a common format

**Context:** All NatSpecs are not formatted properly, lack of @notice, @param, @return

**Description:** It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. [https://docs.soliditylang.org/en/v0.8.15/natspec-format.html](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html)

**Recommendation:** NatSpec comments should be in a common format

## [Q-06] For modern and more readable code; update import usage

**Description:** Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct `polluted the source code` with an unnecessary object we were not using because we did not need it. This was breaking the rule of modularity and modular programming: `only import what you need` Specific imports with curly braces allow us to apply this rule better.

**Recommendation:** `import {contract1 , contract2} from "filename.sol";`

A good example:

```solidity
import {Owned} from "solmate/auth/Owned.sol";
import {ERC721} from "solmate/tokens/ERC721.sol";
import {LibString} from "solmate/utils/LibString.sol";
import {MerkleProofLib} from "solmate/utils/MerkleProofLib.sol";
import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import {ERC1155, ERC1155TokenReceiver} from "solmate/tokens/ERC1155.sol";
import {toWadUnsafe, toDaysWadUnsafe} from "solmate/utils/SignedWadMath.sol";
```

## [Q-07] Assembly Codes Specific – Should Have Comments

Since this is a low-level language that is more difficult to parse readers, include extensive documentation, comments on the rationale behind its use, and clearly explaining what each assembly instruction does

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

```solidity
File: contracts\PositionFactory.sol

function createClone(address target) internal returns (address result) {
    bytes20 targetBytes = bytes20(target);
    assembly {
        let clone := mload(0x40)
        mstore(clone, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
        mstore(add(clone, 0x14), targetBytes)
        mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
        result := create(0, clone, 0x37)
    }
}
```

## [Q-08] Take advantage of Custom Error's return value property

An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the `()` sign, this kind of approach provides a serious advantage in debugging and examining the revert details of dapps such as tenderly.

For Example:

```solidity
File: contracts\Position.sol

if(_coll < minimumCollateral) revert InsufficientCollateral();
if (price > _price) revert InsufficientCollateral();
if (block.timestamp >= start) revert TooLate();
if (collateralReserve * atPrice < minted * ONE_DEC18) revert InsufficientCollateral();
if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();
if (block.timestamp > expiration) revert Expired();
if (block.timestamp <= cooldown) revert Hot();
if (challengedAmount > 0) revert Challenged();
if (msg.sender != address(hub)) revert NotHub();
```

## [Q-09] Function Naming suggestions

Proper use of _ as a function name prefix and a common pattern is to prefix internal and private function names with _. This pattern is correctly applied in the Party contracts, however, there are some inconsistencies in the libraries.

```solidity
File: .\\contracts\\Equity.sol

144: function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {
157: function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){
172: function anchorTime() internal view returns (uint64){
225: function canVoteFor(address delegate, address owner) internal view returns (bool) {
266: function calculateSharesInternal(uint256 capitalBefore, uint256 investment) internal view returns (uint256) {

```

```solidity
File: .\\contracts\\ERC20.sol

97: function allowanceInternal(address owner, address spender) internal view virtual returns (uint256) {

```

```solidity
File: .\\contracts\\Frankencoin.sol

102: function allowanceInternal(address owner, address spender) internal view override returns (uint256) {

```

```solidity
File: .\\contracts\\MintingHub.sol

188: function minBid(Challenge storage challenge) internal view returns (uint256) {
287: function returnCollateral(Challenge storage challenge, bool postpone) internal {

```

```solidity
File: .\\contracts\\Ownable.sol

39: function setOwner(address newOwner) internal {
45: function requireOwner(address sender) internal view {

```

```solidity
File: .\\contracts\\Position.sol

169: function collateralBalance() internal view returns (uint256){
193: function mintInternal(address target, uint256 amount, uint256 collateral_) internal {
202: function restrictMinting(uint256 period) internal {
232: function repayInternal(uint256 burnable) internal {
240: function notifyRepaidInternal(uint256 amount) internal {
268: function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {
282: function checkCollateral(uint256 collateralReserve, uint256 atPrice) internal view {
286: function emitUpdate() internal {

```

```solidity
File: .\\contracts\\PositionFactory.sol

37: function createClone(address target) internal returns (address result) {

```

```solidity
File: .\\contracts\\StablecoinBridge.sol

49: function mintInternal(address target, uint256 amount) internal {
67: function burnInternal(address zchfHolder, address target, uint256 amount) internal {

```

## [Q-10] Order of functions not following the Style Guide

Order of Functions; ordering helps readers identify which functions they can call and find the constructor and fallback definitions easier.
But there are contracts in the project that do not comply with this.
Functions should be grouped according to their visibility and ordered:

- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private.

```solidity
File: .\\contracts\\Equity.sol

209: function checkQualified(address sender, address[] calldata helpers) public override view {
220: function delegateVoteTo(address delegate) external {

```

```solidity
File: .\\contracts\\MintingHub.sol

281: function returnPostponedCollateral(address collateral, address target) external {

```

```solidity
File: .\\contracts\\Position.sol

227: function repay(uint256 amount) public onlyOwner {
249: function withdraw(address token, address target, uint256 amount) external onlyOwner {
292: function notifyChallengeStarted(uint256 size) external onlyHub {

```

```solidity
File: .\\contracts\\StablecoinBridge.sol

55: function burn(uint256 amount) external {

```

## [Q-11] Long lines are not suitable for the `Solidity Style Guide`

It is generally recommended that lines in the source code should not exceed 80-120 characters.
Multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the Maximum Line Length section.

```solidity
File: .\\contracts\\Equity.sol

117: // Recipient votes should stay the same, but grow faster in the future, requiring an adjustment of the anchor.
161: voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); // new example anchor is only 21 / 11 = 1 block in the past
268: uint256 newTotalShares = totalShares < 1000 * ONE_DEC18 ? 1000 * ONE_DEC18 : _mulD18(totalShares, _cubicRoot(_divD18(capitalBefore + investment, capitalBefore)));

```

```solidity
File: .\\contracts\\ERC20.sol

53: // Copied from <https://github.com/OpenZeppelin/openzeppelin-contracts/pull/4139/files#diff-fa792f7d08644eebc519dac2c29b00a54afc4c6a76b9ef3bba56c8401fe674f6>

```

```solidity
File: .\\contracts\\ERC20PermitLight.sol

40: // keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),

```

```solidity
File: .\\contracts\\Frankencoin.sol

78: * adds value to the Frankencoin system. Complex proposals should have application periods and applications fees above
79: * the minimum. It is assumed that over time, informal ways to coordinate on new minters emerge. The message parameter
80: * might be useful for initiating further communication. Maybe it contains a link to a website describing the proposed
83: function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {
115: * The minter reserve can be used to cover losses after all else failed and the equity holders have already been wiped out.
169: minterReserveE6 += _amount * _reservePPM; // minter reserve must be kept accurately in order to ensure we can get back to exactly 0
185: * share holders. This can make sense in combination with 'notifyLoss', i.e. when it is the pool share holders that bear the risk
188: * Design rule: Minters calling this method are only allowed to so for tokens amounts they previously minted with the same _reservePPM amount.
190: * For example, if someone minted 50 ZCHF earlier with a 20% reserve requirement (200000 ppm), they got 40 ZCHF and paid
191: * 10 ZCHF into the reserve. Now they want to repay the debt by burning 50 ZCHF. When doing so using this method, 50 ZCHF get
192: * burned and on top of that, 10 ZCHF previously assigned to the minter's reserved are reassigned to the pool share holders.
201: * Under normal circumstances, this is just the reserver requirement multiplied by the amount. However, after a severe loss
217: * The caller is only allowed to use this method for tokens also minted through the caller with the same _reservePPM amount.
219: * Example: the calling contract has previously minted 100 ZCHF with a reserve ratio of 20% (i.e. 200000 ppm). To burn half
220: * of that again, the minter calls burnFrom with a target amount of 50 ZCHF. Assuming that reserves are only 90% covered,
221: * this call will deduct 41 ZCHF from the payer's balance and 9 from the reserve, while reducing the minter reserve by 10.
223: function burnFrom(address payer, uint256 targetTotalBurnAmount, uint32 _reservePPM) external override minterOnly returns (uint256) {
232: * Calculate the amount that is freed when returning amountExcludingReserve given a reserve ratio of reservePPM, taking
235: function calculateFreedAmount(uint256 amountExcludingReserve /* 41 */, uint32 reservePPM /* 20% */) public view returns (uint256){
238: uint256 adjustedReservePPM = currentReserve < minterReserve_ ? reservePPM * currentReserve / minterReserve_ : reservePPM; // 18%
243: * Burns the provided number of tokens plus whatever reserves are associated with that amount given the reserve requirement.
244: * The caller is only allowed to use this method for tokens also minted through the caller with the same _reservePPM amount.
246: * Example: the calling contract has previously minted 100 ZCHF with a reserve ratio of 20% (i.e. 200000 ppm). Now they have
248: * the call to burnWithReserve will burn the 41 plus 9 from the reserve, reducing the outstanding 'debt' of the caller by
251: function burnWithReserve(uint256 _amountExcludingReserve, uint32 _reservePPM) external override minterOnly returns (uint256) {
254: _transfer(address(reserve), msg.sender, freedAmount - _amountExcludingReserve); // collect assigned reserve, maybe less than original reserve

```

```solidity
File: .\\contracts\\IPosition.sol

36: function notifyChallengeSucceeded(address bidder, uint256 bid, uint256 size) external returns (address, uint256, uint256, uint256, uint32);

```

```solidity
File: .\\contracts\\MintingHub.sol

121: * Clones an existing position and immediately tries to mint the specified amount using the given amount of collateral.
122: * This requires an allowance to be set on the collateral contract such that the minting hub can withdraw the collateral.
124: function clonePosition(address position, uint256 _initialCollateral, uint256 _initialMint) public validPos(position) returns (address) {
140: function launchChallenge(address _positionAddr, uint256 _collateralAmount) external validPos(_positionAddr) returns (uint256) {
144: challenges.push(Challenge(msg.sender, position, _collateralAmount, block.timestamp + position.challengePeriod(), address(0x0), 0));
247: * In case that the collateral cannot be transfered back to the challenger (i.e. because the collateral token has a blacklist and the
250: * @param postponeCollateralReturn Can be used to postpone the return of the collateral to the challenger. Usually false.
256: // challenge must have been successful, because otherwise it would have immediately ended on placing the winning bid
258: // notify the position that will send the collateral to the bidder. If there is no bid, send the collateral to msg.sender
260: (address owner, uint256 effectiveBid, uint256 volume, uint256 repayment, uint32 reservePPM) = challenge.position.notifyChallengeSucceeded(recipient, challenge.bid, challenge.size);
289: // Postponing helps in case the challenger was blacklisted on the collateral token or otherwise cannot receive it at the moment.
294: challenge.position.collateral().transfer(challenge.challenger, challenge.size); // return the challenger's collateral

```

```solidity
File: .\\contracts\\Position.sol

21: uint256 public price; // the zchf price per unit of the collateral below which challenges succeed, (36 - collateral.decimals) decimals
76: function initializeClone(address owner, uint256 _price, uint256 _limit, uint256 _coll, uint256 _mint) external onlyHub {
213: * The repaid amount should fulfill the following equation in order to close the position, i.e. bring the minted amount to 0:
219: * For example, if minted is 50 and reservePPM is 200000, it is necessary to repay 40 to be able to close the position.
221: * Only the owner is allowed to repay a position. This is necessary to prevent a 'limit stealing attack': if a popular position
222: * has reached its limit, an attacker could try to repay the position, clone it, and take a loan himself. This is prevented by
223: * requiring the owner to do the repayment. Other restrictions are not necessary. In particular, it must be possible to repay
224: * the position once it is expired or subject to cooldown. Also, repaying it during a challenge is no problem as the collateral
327: * @return (position owner, effective bid size in ZCHF, effective challenge size in ZCHF, repaid amount, reserve ppm)
329: function notifyChallengeSucceeded(address _bidder, uint256 _bid, uint256 _size) external onlyHub returns (address, uint256, uint256, uint256, uint32) {
347: uint256 volumeZCHF = _mulD18(price, _size); // How much could have minted with the challenged amount of the collateral

```

```solidity
File: .\\contracts\\PositionFactory.sol

36: // copied from <https://github.com/optionality/clone-factory/blob/32782f82dfc5a00d103a7e61a17a5dedbd1e8e9d/contracts/CloneFactory.sol>

```

## [Q-12] **Floating pragma**

Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

[https://swcregistry.io/docs/SWC-103](https://swcregistry.io/docs/SWC-103)

```solidity
All Contracts
```