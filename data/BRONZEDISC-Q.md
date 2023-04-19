## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol

```solidity
// modifier coming after all functions. Should come before all of them since there is no constructor
49:    modifier onlyOwner() {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

```solidity
// error declarations should come before constructor
92:   error PeriodTooShort();
93:   error FeeTooLow();
94:   error AlreadyRegistered();
130:   error NotMinter();
159:   error TooLate();

// modifier should come before constructor
266:   modifier minterOnly() {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

```solidity
// error declarations should come before constructor
214:    error NotQualified();
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

```solidity
// modifiers should come before constructor as described above.
115:    modifier validPos(address position) {

// error declarations should come before constructor as described above.
231:    error TooLate();
232:    error UnexpectedSize();
233:    error BidTooLow(uint256 bid, uint256 min);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

```solidity
// errors should all come before constructor for a better organization
103:    error TooLate();
104:    error NotQualified();
191:    error LimitExceeded();
238:    error RepaidTooMuch(uint256 excess);
290:    error ChallengeTooSmall();
364:    error Expired();
371:    error Hot();
378:    error Challenged();
385:    error NotHub();

// modifiers should come before constructor for a better organization
366:    modifier alive() {
373:    modifier noCooldown() {
380:    modifier noChallenge() {
387:    modifier onlyHub() {
```
---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), public, external, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol

```solidity
// internal functions coming before external one
49:    function mintInternal(address target, uint256 amount) internal {
67:    function burnInternal(address zchfHolder, address target, uint256 amount) internal {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol

```solidity
// internal functions coming before external ones
97:    function allowanceInternal(address owner, address spender) internal view virtual returns (uint256) {
151:    function _transfer(address sender, address recipient, uint256 amount) internal virtual {

// move the view/pure functions to the last of their respective categories
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

```solidity
// public functions coming after external and some internals
117:   function minterReserve() public view returns (uint256) {
204:   function calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) public view returns (uint256) {
235:   function calculateFreedAmount(uint256 amountExcludingReserve /* 41 */, uint32 reservePPM /* 20% */) public view returns (uint256){
293:   function isMinter(address _minter) override public view returns (bool){
300:   function isPosition(address _position) override public view returns (address){

// internal function coming before public and external ones. should come last since there are not privates on this code.
102:   function allowanceInternal(address owner, address spender) internal view override returns (uint256) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

```solidity
// some external functions are coming before public and internal functions. order them as listed above
97:    function name() override external pure returns (string memory) {
101:    function symbol() override external pure returns (string memory) {
127:    function canRedeem() external view returns (bool){
220:    function delegateVoteTo(address delegate) external {
241:    function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool) {

// some internal functions are coming before public and external functions. order them as listed above
112:    function _beforeTokenTransfer(address from, address to, uint256 amount) override internal {
144:    function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {
157:    function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){
172:    function anchorTime() internal view returns (uint64){
266:    function calculateSharesInternal(uint256 capitalBefore, uint256 investment) internal view returns (uint256) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

```solidity
// public functions coming after external and some internal ones.
181:    function minBid(uint256 challenge) public view returns (uint256) {
252:    function end(uint256 _challengeNumber, bool postponeCollateralReturn) public {

// internal fn coming before public and external ones.
188:    function minBid(Challenge storage challenge) internal view returns (uint256) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

```solidity
// these public functions are coming after some external ones, they should all come before
109:    function deny(address[] calldata helpers, string calldata message) public {
120:    function getUsableMint(uint256 totalMint, bool afterFees) public view returns (uint256){
132:    function adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) public onlyOwner {
159:    function adjustPrice(uint256 newPrice) public onlyOwner noChallenge {
177:    function mint(address target, uint256 amount) public onlyOwner noChallenge noCooldown alive {
181:    function calculateCurrentFee() public view returns (uint32) {
227:    function repay(uint256 amount) public onlyOwner {
263:    function withdrawCollateral(address target, uint256 amount) public onlyOwner noChallenge noCooldown {
360:    function isClosed() public view returns (bool) {

// these internal functions are coming before some public and external ones.they should come 2last since there is not private functions in this file
169:    function collateralBalance() internal view returns (uint256){
193:    function mintInternal(address target, uint256 amount, uint256 collateral_) internal {
202:    function restrictMinting(uint256 period) internal {
232:    function repayInternal(uint256 burnable) internal {
240:    function notifyRepaidInternal(uint256 amount) internal {
268:    function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {
282:    function checkCollateral(uint256 collateralReserve, uint256 atPrice) internal view {
286:    function emitUpdate() internal {
```
---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol

```solidity
23:    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

25:    error NotOwner();

// @param missing
31:    function transferOwnership(address newOwner) public onlyOwner {
39:    function setOwner(address newOwner) internal {

45:    function requireOwner(address sender) internal view {

49:    modifier onlyOwner() {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol

```solidity
31:    function _mulD18(uint256 _a, uint256 _b) internal pure returns(uint256) {

35:    function _divD18(uint256 _a, uint256 _b) internal pure returns(uint256) {

39:    function _power3(uint256 _x) internal pure returns(uint256) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol

```solidity
7:    contract PositionFactory {

// not natSpec compliant
13:    function createNewPosition(address _owner, address _zchf, address _collateral, uint256 _minCollateral, 
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol

```solidity
26:    constructor(address other, address zchfAddress, uint256 limit_){

// not nasSpec compliant
36:    function mint(uint256 amount) external {
44:    function mint(address target, uint256 amount) public {
63:    function burn(address target, uint256 amount) external {
75:    function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool){

49:    function mintInternal(address target, uint256 amount) internal {
55:    function burn(uint256 amount) external {
67:    function burnInternal(address zchfHolder, address target, uint256 amount) internal {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol

```solidity
21:    function permit(

61:    function DOMAIN_SEPARATOR() public view returns (bytes32) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol

```solidity
59:    constructor(uint8 _decimals) {

// not natSpec compliant
85:    function transfer(address recipient, uint256 amount) public virtual override returns (bool) {

97:    function allowanceInternal(address owner, address spender) internal view virtual returns (uint256) {

162:    function transferAndCall(address recipient, uint256 amount, bytes calldata data) external override returns (bool) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

```solidity
// not natSpec compliant
59:   constructor(uint256 _minApplicationPeriod) ERC20(18){
83:   function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {
102:   function allowanceInternal(address owner, address spender) internal view override returns (uint256) {
125:   function registerPosition(address _position) override external {
138:   function equity() public view returns (uint256) {
152:   function denyMinter(address _minter, address[] calldata _helpers, string calldata _message) override external {
165:   function mint(address _target, uint256 _amount, uint32 _reservePPM, uint32 _feesPPM) override external minterOnly {
179:   function burn(uint256 _amount) external {
194:   function burn(uint256 amount, uint32 reservePPM) external override minterOnly {
204:   function calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) public view returns (uint256) {
223:   function burnFrom(address payer, uint256 targetTotalBurnAmount, uint32 _reservePPM) external override minterOnly returns (uint256) {
235:   function calculateFreedAmount(uint256 amountExcludingReserve /* 41 */, uint32 reservePPM /* 20% */) public view returns (uint256){
251:   function burnWithReserve(uint256 _amountExcludingReserve, uint32 _reservePPM) external override minterOnly returns (uint256) {
262:   function burn(address _owner, uint256 _amount) override external minterOnly {
280:   function notifyLoss(uint256 _amount) override external minterOnly {
293:   function isMinter(address _minter) override public view returns (bool){
300:   function isPosition(address _position) override public view returns (address){

64:   function name() override external pure returns (string memory){

68:   function symbol() override external pure returns (string memory){

// @return missing
117:   function minterReserve() public view returns (uint256) {

172:   function mint(address _target, uint256 _amount) override external minterOnly {

266:   modifier minterOnly() {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

```solidity
90:    event Delegation(address indexed from, address indexed to); // indicates a delegation
91:    event Trade(address who, int amount, uint totPrice, uint newprice); // amount pos or neg for mint or redemption

93:    constructor(Frankencoin zchf_) ERC20(18) {

97:    function name() override external pure returns (string memory) {

101:    function symbol() override external pure returns (string memory) {

// not natSpec compliant
108:    function price() public view returns (uint256){
127:    function canRedeem() external view returns (bool){
135:    function canRedeem(address owner) public view returns (bool) {
172:    function anchorTime() internal view returns (uint64){
179:    function votes(address holder) public view returns (uint256) {
186:    function totalVotes() public view returns (uint256) {
209:    function checkQualified(address sender, address[] calldata helpers) public override view {
220:    function delegateVoteTo(address delegate) external {
241:    function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool) {

112:    function _beforeTokenTransfer(address from, address to, uint256 amount) override internal {

190:    function votes(address sender, address[] calldata helpers) public view returns (uint256) {

225:    function canVoteFor(address delegate, address owner) internal view returns (bool) {

266:    function calculateSharesInternal(uint256 capitalBefore, uint256 investment) internal view returns (uint256) {

275:    function redeem(address target, uint256 shares) public returns (uint256) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

```solidity
48:    event ChallengeStarted(address indexed challenger, address indexed position, uint256 size, uint256 number);
49:    event ChallengeAverted(address indexed position, uint256 number);
50:    event ChallengeSucceeded(address indexed position, uint256 bid, uint256 number);
51:    event NewBid(uint256 challengedId, uint256 bidAmount, address bidder);
52:    event PostPonedReturn(address collateral, address indexed beneficiary, uint256 amount);

54:    constructor(address _zchf, address factory) {

59:    function openPosition(

115:    modifier validPos(address position) {

// not natSpec format
124:    function clonePosition(address position, uint256 _initialCollateral, uint256 _initialMint) public validPos(position) returns (address) {
156:    function splitChallenge(uint256 _challengeNumber, uint256 splitOffAmount) external returns (uint256) {
188:    function minBid(Challenge storage challenge) internal view returns (uint256) {

181:    function minBid(uint256 challenge) public view returns (uint256) {

235:    function end(uint256 _challengeNumber) external {

239:    function isChallengeOpen(uint256 _challengeNumber) external view returns (bool) {

// @param missing
252:    function end(uint256 _challengeNumber, bool postponeCollateralReturn) public {

281:    function returnPostponedCollateral(address collateral, address target) external {

287:    function returnCollateral(Challenge storage challenge, bool postpone) internal {

299:    interface IPositionFactory {

300:    function createNewPosition(

314:    function clonePosition(address _existing) external returns (address);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

```solidity
41:    event PositionOpened(address indexed owner, address original, address zchf, address collateral, uint256 price);

42:    event MintingUpdate(uint256 collateral, uint256 price, uint256 minted, uint256 limit);

43:    event PositionDenied(address indexed sender, string message); // emitted if closed by governance

50:    constructor(address _owner, address _hub, address _zchf, address _collateral, 

76:    function initializeClone(address owner, uint256 _price, uint256 _limit, uint256 _coll, uint256 _mint) external onlyHub {

109:    function deny(address[] calldata helpers, string calldata message) public {

120:    function getUsableMint(uint256 totalMint, bool afterFees) public view returns (uint256){

132:    function adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) public onlyOwner {

159:    function adjustPrice(uint256 newPrice) public onlyOwner noChallenge {

169:    function collateralBalance() internal view returns (uint256){

177:    function mint(address target, uint256 amount) public onlyOwner noChallenge noCooldown alive {

181:    function calculateCurrentFee() public view returns (uint32) {

193:    function mintInternal(address target, uint256 amount, uint256 collateral_) internal {

202:    function restrictMinting(uint256 period) internal {

// @param missing
227:    function repay(uint256 amount) public onlyOwner {

232:    function repayInternal(uint256 burnable) internal {

240:    function notifyRepaidInternal(uint256 amount) internal {

249:    function withdraw(address token, address target, uint256 amount) external onlyOwner {

// @params missing
263:    function withdrawCollateral(address target, uint256 amount) public onlyOwner noChallenge noCooldown {

268:    function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {

282:    function checkCollateral(uint256 collateralReserve, uint256 atPrice) internal view {

286:    function emitUpdate() internal {

292:    function notifyChallengeStarted(uint256 size) external onlyHub {

// @return missing
360:    function isClosed() public view returns (bool) {

366:    modifier alive() {

373:    modifier noCooldown() {

380:    modifier noChallenge() {

387:    modifier onlyHub() {
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol

```solidity
// private and internal `functions` should preppend with `underline`
39:    function setOwner(address newOwner) internal {
45:    function requireOwner(address sender) internal view {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol

```solidity
// private and internal `functions` should preppend with `underline`
37:    function createClone(address target) internal returns (address result) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol

```solidity
// private and internal `functions` should preppend with `underline`
49:    function mintInternal(address target, uint256 amount) internal {
67:    function burnInternal(address zchfHolder, address target, uint256 amount) internal {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol

```solidity
//  private and internal `functions` should preppend with `underline`
97:    function allowanceInternal(address owner, address spender) internal view virtual returns (uint256) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

```solidity
// private and internal `variables` should preppend with `underline`
39:   uint256 private minterReserveE6;

// private and internal `functions` should preppend with `underline`
102:   function allowanceInternal(address owner, address spender) internal view override returns (uint256) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

```solidity
// private and internal `variables` should preppend with `underline`
75:    uint192 private totalVotesAtAnchor;  // Total number of votes at the anchor time, see comment on the um
76:    uint64 private totalVotesAnchorTime; // 40 Bit for the block number, 24 Bit sub-block time resolution
88:    mapping (address => uint64) private voteAnchor; // 40 Bit for the block number, 24 Bit sub-block time resolution

//private and internal `functions` should preppend with `underline`
144:    function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {
157:    function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){
172:    function anchorTime() internal view returns (uint64){
225:    function canVoteFor(address delegate, address owner) internal view returns (bool) {
266:    function calculateSharesInternal(uint256 capitalBefore, uint256 investment) internal view returns (uint256) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

```solidity
188:    function minBid(Challenge storage challenge) internal view returns (uint256) {
287:    function returnCollateral(Challenge storage challenge, bool postpone) internal {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

```solidity
// private and internal `functions` should preppend with `underline`
169:    function collateralBalance() internal view returns (uint256){
193:    function mintInternal(address target, uint256 amount, uint256 collateral_) internal {
202:    function restrictMinting(uint256 period) internal {
232:    function repayInternal(uint256 burnable) internal {
240:    function notifyRepaidInternal(uint256 amount) internal {
268:    function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {
282:    function checkCollateral(uint256 collateralReserve, uint256 atPrice) internal view {
286:    function emitUpdate() internal {
```

---

### Version [5]

- Pragma versions should be standardized and avoid floating pragma `( ^ )`.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol

```solidity
// lose the ^ to lock the version making it safer
9:    pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol

```solidity
// lose the ^ to lock the version making it safer
3:    pragma solidity >=0.8.0 <0.9.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol

```solidity
// lose the ^ to lock the version making it safer
2:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol

```solidity
// lose the ^ to lock the version making it safer
2:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol

```solidity
// lose the ^ to lock the version making it safer
5:   pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol

```solidity
// lose the ^ to lock the version making it safer
12:   pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

```solidity
// lose the ^ to lock the version making it safer
2:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

```solidity
// lose the ^ to lock the version making it safer
4:    pragma solidity >=0.8.0 <0.9.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

```solidity
// lose the ^ to lock the version making it safer
2:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

```solidity
// lose the ^ to lock the version making it safer
2:  pragma solidity ^0.8.0;
```


---

### Observations [6]

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol

```solidity
// this function is not adding underscore to the param name but the rest of the functions are. pick one of these patterns and stick to it.
37:    function createClone(address target) internal returns (address result) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol

```solidity
// this function is adding underscore to the param name but the rest of the functions are not. pick one of these patterns and stick to it. Some are mixing both styles. 
59:    constructor(uint8 _decimals) {

// move the `virtual` keyword to the end of the line
240:    function _beforeTokenTransfer(address from, address to, uint256 amount) virtual internal {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

```solidity
// change `override` to the end of the line right before `returns` if there is one.
64:   function name() override external pure returns (string memory){
68:   function symbol() override external pure returns (string memory){
83:   function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {
125:   function registerPosition(address _position) override external {
152:   function denyMinter(address _minter, address[] calldata _helpers, string calldata _message) override external {
165:   function mint(address _target, uint256 _amount, uint32 _reservePPM, uint32 _feesPPM) override external minterOnly {
172:   function mint(address _target, uint256 _amount) override external minterOnly {
262:   function burn(address _owner, uint256 _amount) override external minterOnly {
280:   function notifyLoss(uint256 _amount) override external minterOnly {
293:   function isMinter(address _minter) override public view returns (bool){
300:   function isPosition(address _position) override public view returns (address){

// these functions are not adding underscore to the params names but the rest of the functions are. pick one these patterns and stick to it. Some are mixing both styles. 
102:   function allowanceInternal(address owner, address spender) internal view override returns (uint256) {
194:   function burn(uint256 amount, uint32 reservePPM) external override minterOnly {
204:   function calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) public view returns (uint256) {
223:   function burnFrom(address payer, uint256 targetTotalBurnAmount, uint32 _reservePPM) external override minterOnly returns (uint256) {
235:   function calculateFreedAmount(uint256 amountExcludingReserve /* 41 */, uint32 reservePPM /* 20% */) public view returns (uint256){
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

```solidity
// change `override` to the end of the line right before `returns` if there is one.
97:    function name() override external pure returns (string memory) {
101:    function symbol() override external pure returns (string memory) {
112:    function _beforeTokenTransfer(address from, address to, uint256 amount) override internal {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

```solidity
// these functions are adding underscore to the params names but the rest of the functions are not. pick one these patterns and stick to it. Some are mixing both styles. 
54:    constructor(address _zchf, address factory) {
124:    function clonePosition(address position, uint256 _initialCollateral, uint256 _initialMint) public validPos(position) returns (address) {
156:    function splitChallenge(uint256 _challengeNumber, uint256 splitOffAmount) external returns (uint256) {
199:    function bid(uint256 _challengeNumber, uint256 _bidAmountZCHF, uint256 expectedSize) external {
235:    function end(uint256 _challengeNumber) external {
239:    function isChallengeOpen(uint256 _challengeNumber) external view returns (bool) {
252:    function end(uint256 _challengeNumber, bool postponeCollateralReturn) public {
300:    function createNewPosition(
314:    function clonePosition(address _existing) external returns (address);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

```solidity
// these functions are adding underscore to the params names but the rest of the functions are not. pick one these patterns and stick to it. the first function even mix both styles. 
76:    function initializeClone(address owner, uint256 _price, uint256 _limit, uint256 _coll, uint256 _mint) external onlyHub {
97:    function reduceLimitForClone(uint256 _minimum) external noChallenge noCooldown alive onlyHub returns (uint256) {
304:    function tryAvertChallenge(uint256 _collateralAmount, uint256 _bidAmountZCHF) external onlyHub returns (bool) {
329:    function notifyChallengeSucceeded(address _bidder, uint256 _bid, uint256 _size) external onlyHub returns (address, uint256, uint256, uint256, uint32) {
```
