# Report
## Low Risk ##
### [L-1]: Unsafe casting may overflow
**Context:**

1. ```totalVotesAtAnchor = uint192(totalVotes() - roundingLoss - lostVotes);``` [L146](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L146)
1. ```voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); // new example anchor is only 21 / 11 = 1 block in the past``` [L161](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L161)

**Description:**

While Solidity 0.8.x checks for overflows on arithmetic operations, it does not do so for casting.

**Recommendation:**

Use OpenZeppelin’s SafeCast library to prevent unexpected overflows.

## Non-Critical Issues ##
### [N-1]: Follow Solidity standard naming conventions
**Context:**

1. ```IPositionFactory private immutable POSITION_FACTORY; // position contract to clone``` [L28](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L28) (Local and State Variable should use mixedCase)
1. ```uint256 public immutable MIN_APPLICATION_PERIOD; // for example 10 days``` [L26](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L26) (Local and State Variable should use mixedCase)
1. ```function DOMAIN_SEPARATOR() public view returns (bytes32) {``` [L61](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L61) (Functions should use mixedCase)

**Description:**

The above codes don't follow [Solidity's standard naming convention](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions).  
- Structs should be named using the CapWords style. Examples: MyCoin, Position, PositionXY.
- Events should be named using the CapWords style. Examples: Deposit, Transfer, Approval, BeforeTransfer, AfterTransfer.
- Functions should use mixedCase. Examples: getBalance, transfer, verifyOwner, addMember, changeOwner.
- Function arguments should use mixedCase. Examples: initialSupply, account, recipientAddress, senderAddress, newOwner.
- Local and State Variable should use mixedCase. Examples: totalSupply, remainingSupply, balancesOf, creatorAddress, isPreSale, tokenExchangeRate.
- Constants should be named with all capital letters with underscores separating words. Examples: MAX_BLOCKS, TOKEN_NAME, TOKEN_TICKER, CONTRACT_VERSION.
- Modifier should use mixedCase. Examples: onlyBy, onlyAfter, onlyDuringThePreSale.
- Enums, in the style of simple type declarations, should be named using the CapWords style. Examples: TokenGroup, Frame, HashStyle, CharacterLocation.

### [N-2]: Missing leading underscores
**Context:**

1. ```function collateralBalance() internal view returns (uint256){``` [L169](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L169) 
1. ```function mintInternal(address target, uint256 amount, uint256 collateral_) internal {``` [L193](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L193) 
1. ```function restrictMinting(uint256 period) internal {``` [L202](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L202) 
1. ```function repayInternal(uint256 burnable) internal {``` [L232](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L232) 
1. ```function notifyRepaidInternal(uint256 amount) internal {``` [L240](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L240) 
1. ```function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {``` [L268](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L268) 
1. ```function checkCollateral(uint256 collateralReserve, uint256 atPrice) internal view {``` [L282](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L282) 
1. ```function emitUpdate() internal {``` [L286](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L286) 
1. ```IPositionFactory private immutable POSITION_FACTORY; // position contract to clone``` [L28](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L28) 
1. ```function minBid(Challenge storage challenge) internal view returns (uint256) {``` [L188](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L188) 
1. ```function returnCollateral(Challenge storage challenge, bool postpone) internal {``` [L287](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L287) 
1. ```uint256 private minterReserveE6;``` [L39](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L39) 
1. ```function allowanceInternal(address owner, address spender) internal view override returns (uint256) {``` [L102](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L102) 
1. ```function mintInternal(address target, uint256 amount) internal {``` [L49](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L49) 
1. ```function burnInternal(address zchfHolder, address target, uint256 amount) internal {``` [L67](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L67) 
1. ```function createClone(address target) internal returns (address result) {``` [L37](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L37) 
1. ```uint256 internal constant ONE_DEC18 = 10**18;``` [L10](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L10) 
1. ```uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01``` [L11](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L11) 
1. ```function setOwner(address newOwner) internal {``` [L39](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L39) 
1. ```function requireOwner(address sender) internal view {``` [L45](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L45) 
1. ```uint256 private constant MINIMUM_EQUITY = 1000 * ONE_DEC18;``` [L41](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L41) 
1. ```uint32 private constant QUORUM = 300;``` [L46](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L46) 
1. ```uint8 private constant BLOCK_TIME_RESOLUTION_BITS = 24;``` [L51](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L51) 
1. ```uint192 private totalVotesAtAnchor;  // Total number of votes at the anchor time, see comment on the um``` [L75](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L75) 
1. ```uint64 private totalVotesAnchorTime; // 40 Bit for the block number, 24 Bit sub-block time resolution``` [L76](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L76) 
1. ```mapping (address => uint64) private voteAnchor; // 40 Bit for the block number, 24 Bit sub-block time resolution``` [L88](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L88) 
1. ```function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {``` [L144](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L144) 
1. ```function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){``` [L157](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L157) 
1. ```function anchorTime() internal view returns (uint64){``` [L172](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L172) 
1. ```function canVoteFor(address delegate, address owner) internal view returns (bool) {``` [L225](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L225) 
1. ```function calculateSharesInternal(uint256 capitalBefore, uint256 investment) internal view returns (uint256) {``` [L266](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L266) 
1. ```uint256 internal constant INFINITY = (1 << 255);``` [L47](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L47) 
1. ```function allowanceInternal(address owner, address spender) internal view virtual returns (uint256) {``` [L97](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L97) 

**Description:**

Internal and private functions, state variables, constants, and immutables should starting with an underscore.

### [N-3]: Recommendations for long function declarations
**Context:**

1. https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L50-L52 
1. ```function initializeClone(address owner, uint256 _price, uint256 _limit, uint256 _coll, uint256 _mint) external onlyHub {``` [L76](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L76) 
1. ```function notifyChallengeSucceeded(address _bidder, uint256 _bid, uint256 _size) external onlyHub returns (address, uint256, uint256, uint256, uint32) {``` [L329](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L329) 
1. https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L60-L62 
1. https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L89-L91 
1. ```function clonePosition(address position, uint256 _initialCollateral, uint256 _initialMint) public validPos(position) returns (address) {``` [L124](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L124) 
1. ```function launchChallenge(address _positionAddr, uint256 _collateralAmount) external validPos(_positionAddr) returns (uint256) {``` [L140](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L140) 
1. ```function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {``` [L83](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L83) 
1. ```function burnFrom(address payer, uint256 targetTotalBurnAmount, uint32 _reservePPM) external override minterOnly returns (uint256) {``` [L223](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L223) 
1. ```function calculateFreedAmount(uint256 amountExcludingReserve /* 41 */, uint32 reservePPM /* 20% */) public view returns (uint256){``` [L235](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L235) 
1. ```function burnWithReserve(uint256 _amountExcludingReserve, uint32 _reservePPM) external override minterOnly returns (uint256) {``` [L251](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L251) 

**Description:**

According to [official solidity documentation](https://docs.soliditylang.org/en/v0.8.19/style-guide.html#function-declaration):
1. For long function declarations, it is recommended to drop each argument onto its own line at the same indentation level as the function body. The closing parenthesis and opening bracket should be placed on their own line as well at the same indentation level as the function declaration.
1. If a long function declaration has modifiers, then each modifier should be dropped to its own line.

### [N-4]: Multiline output parameters style
**Context:**

1. ```function notifyChallengeSucceeded(address _bidder, uint256 _bid, uint256 _size) external onlyHub returns (address, uint256, uint256, uint256, uint32) {``` [L329](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L329) 

**Description:**

According to [official solidity documentation](https://docs.soliditylang.org/en/v0.8.19/style-guide.html#function-declaration) multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the [Maximum Line Length section](https://docs.soliditylang.org/en/v0.8.19/style-guide.html#maximum-line-length).

**Recommendation:**

Example:  
```
function thisFunctionNameIsReallyLong(
    address a,
    address b,
    address c
)
    public
    returns (
        address someAddressName,
        uint256 LongArgument,
        uint256 Argument
    )
{
    doSomething()

    return (
        veryLongReturnArg1,
        veryLongReturnArg2,
        veryLongReturnArg3
    );
}
```

### [N-5]: Wrong order of layout
**Context:**

1. ```modifier alive() {``` [L366](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L366) (modifiers must be before all functions)
1. ```modifier noCooldown() {``` [L373](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L373) (modifiers must be before all functions)
1. ```modifier noChallenge() {``` [L380](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L380) (modifiers must be before all functions)
1. ```modifier onlyHub() {``` [L387](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L387) (modifiers must be before all functions)
1. ```modifier validPos(address position) {``` [L115](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L115) (modifiers must be before all functions)
1. ```modifier minterOnly() {``` [L266](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L266) (modifiers must be before all functions)
1. ```modifier onlyOwner() {``` [L49](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L49) (modifiers must be before all functions)

**Description:**
According to [official solidity documentation](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout) inside each contract, library or interface, use the following order:
1) Type declarations
2) State variables
3) Events
4) Modifiers
5) Functions

### [N-6]: Natspec is incomplete
**Context:**

1. ```function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {``` [L144](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L144) (param roundingLoss is missing)
