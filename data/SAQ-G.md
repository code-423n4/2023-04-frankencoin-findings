## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Functions guaranteed to revert when called by normal users can be marked payable | 12 | - |
| [G-02] | Do not calculate constants | 3 | - |
| [G-03] | ++i Costs Less Gas Than i++, Especially When It’s Used In For-loops (--i/i-- Too) | 1 | - |
| [G-04] | internal functions only called once can be inlined to save gas | 2 | - |
| [G-05] | Public Functions To External | 5 | - |
| [G-06] | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= )| 17 | - |
| [G-07] .|++i/i++ SHOULD BE UNCHECKED{++i}/UNCHECKED{i++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN  FOR  AND WHILE-LOOPS | 3 | - |
| [G-08] | MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE      APPROPRIATE | 4 | - |
| [G-09] | SHOULD USE ARGUMENTS INSTEAD OF STATE VARIABLE | 26 | - |
| [G-10] | USE ASSEMBLY TO CHECK FOR ADDRESS(0)  | 3 | - |
| [G-11] | Use nested if and, avoid multiple check  | 4 | - |




## Gas Optimizations  

## [G-1] Functions guaranteed to revert when called by normal users can be marked `payable`

### Details

>If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay >the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not >include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2>>>>(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the >function, in addition to the extra deployment cost.

```solidity
file:  /contracts/Position.sol
76   function initializeClone(address owner, uint256 _price, uint256 _limit, uint256 _coll, uint256 _mint) external onlyHub {

97     function reduceLimitForClone(uint256 _minimum) external noChallenge noCooldown alive onlyHub returns (uint256) {

132    function adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) public onlyOwner {

159    function adjustPrice(uint256 newPrice) public onlyOwner noChallenge {  

177    function mint(address target, uint256 amount) public onlyOwner noChallenge noCooldown alive {

227    function repay(uint256 amount) public onlyOwner {

249    function withdraw(address token, address target, uint256 amount) external onlyOwner {  

263    function withdrawCollateral(address target, uint256 amount) public onlyOwner noChallenge noCooldown {

292    function notifyChallengeStarted(uint256 size) external onlyHub {

304    function tryAvertChallenge(uint256 _collateralAmount, uint256 _bidAmountZCHF) external onlyHub returns (bool) {

329    function notifyChallengeSucceeded(address _bidder, uint256 _bid, uint256 _size) external onlyHub returns (address, uint256, uint256, uint256, uint32) {  
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

```solidity
File: /contracts/Ownable.sol

31    function transferOwnership(address newOwner) public onlyOwner {
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L31

## [G-2] Do not calculate `constants`

### Details

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```solidity
file: /contracts/Equity.sol
39 uint32 public constant VALUATION_FACTOR = 3;

46   uint32 private constant QUORUM = 300;

51     uint8 private constant BLOCK_TIME_RESOLUTION_BITS = 24;

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

## [G-3]  ++i Costs Less Gas Than i++, Especially When It’s Used In For-loops (--i/i-- Too)

```solidity
file: /contracts/Equity.sol
196 for (uint j=i+1; j<helpers.length; j++)

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L196

## [G-4] `internal` functions only called once can be `inlined` to save gas

```solidity
file: /contracts/MathUtil.sol
18   function _cubicRoot(uint256 _v) internal pure returns (uint256) {
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L18


```solidity
file:/contracts/Equity.sol
157  function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L157

## [G-5] `Public` Functions To `External`

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

```solidity
file: /contracts/MintingHub.sol
88 function openPosition(
        address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
        uint256 _mintingMaximum, uint256 _initPeriodSeconds, uint256 _expirationSeconds, uint256 _challengeSeconds,
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address) {

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L88


```solidity
file: /contracts/MintingHub.sol
252    function end(uint256 _challengeNumber, bool postponeCollateralReturn) public {

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L252


```solidity
file:/contracts/Equity.sol
135   function canRedeem(address owner) public view returns (bool) {

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L135


```solidity
file: /contracts/Equity.sol
190    function votes(address sender, address[] calldata helpers) public view returns (uint256) {

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L190


```solidity
file:/contracts/StablecoinBridge.sol
44     function mint(address target, uint256 amount) public {

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L44


## [G-6] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES or ( -= )

### Detials
 AVOID `COMPOUND ASSIGNMENT` OPERATOR IN `STATE VARIABLES`

Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).

```solidity
file: /contracts/Position.sol
295   challengedAmount += size;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L295

```solidity
file: /contracts/Position.sol
309   challengedAmount -= _collateralAmount;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L309

```solidity
file:/contracts/Position.sol
330  challengedAmount -= _size;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L330

```solidity
file:/contracts/Position.sol
99  limit -= reduction + _minimum;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L99

```solidity
file:/contracts/Position.sol
196 minted += amount;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L196

```solidity
file:/contracts/Position.sol
242 minted -= amount;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L242

```solidity
file:/contracts/Position.sol
99  challengedAmount -= _collateralAmount;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L99

```solidity
file:/contracts/Frankencoin.sol
169   minterReserveE6 += _amount * _reservePPM; // minter reserve must be kept accurately in order to ensure we can get back to exactly 0
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L169

```solidity
file: /contracts/Frankencoin.sol
196 minterReserveE6 -= amount * reservePPM;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L196

```solidity
file: /contracts/Frankencoin.sol
227  minterReserveE6 -= targetTotalBurnAmount * _reservePPM; // reduce reserve requirements by original ratio

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L227

```solidity
file:/contracts/Frankencoin.sol
253  minterReserveE6 -= freedAmount * _reservePPM; // reduce reserve requirements by original ratio

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L253

 ```solidity
file:/contracts/ERC20.sol
156    _balances[sender] -= amount;

157    _balances[recipient] += amount;

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L156-L157

```solidity
file:/contracts/ERC20.sol
185 _balances[recipient] += amount;

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L185

```solidity
file:/contracts/ERC20.sol
204   _balances[account] -= amount;

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L204

```solidity
file: /contracts/ERC20.sol
184  _totalSupply += amount;

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L184

```solidity
file: /contracts/ERC20.sol
203 _totalSupply -= amount;

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L203

## [G-7] ++i/i++ SHOULD BE UNCHECKED{++i}/UNCHECKED{i++} WHEN IT IS NOT POSSIBLE FOR THEM TO OVERFLOW, AS IS THE CASE WHEN USED IN FOR- AND WHILE-LOOPS

### Details

In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each 
iteration, but at the cost of some code readability, as this uncheck cannot be made inline..

#### Summary
Unchecked operations as the ++i on for loops are cheaper than checked one.


```solidity
file:/contracts/Equity.sol
192  for (uint i=0; i<helpers.length; i++){
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L192


```solidity
file:/contracts/Equity.sol
196  for (uint j=i+1; j<helpers.length; j++){

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L196


```solidity
file:/contracts/Equity.sol
312   for (uint256 i = 0; i<addressesToWipe.length; i++){

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L312


## [G-8] MULTIPLE ADDRESS/ID MAPPINGS CAN BE COMBINED INTO A SINGLE MAPPING OF AN ADDRESS/ID TO A STRUCT, WHERE APPROPRIATE

### Details

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to re
 the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.


```solidity
file:/contracts/Equity.sol
83  mapping (address => address) public delegates;

88  mapping (address => uint64) private voteAnchor; // 40 Bit for the block number, 24 Bit sub-block time resolution

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L83-L88


```solidity
file:/contracts/Frankencoin.sol
45  mapping (address => uint256) public minters;

50  mapping (address => address) public positions;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L45-L50


## [G-9] SHOULD USE ARGUMENTS INSTEAD OF STATE VARIABLE


```solidity
file:/contracts/Position.sol
80   price = _mint * ONE_DEC18 / _coll;

143   minted = newMinted;

196   minted += amount;

242   minted -= amount;

295     challengedAmount += size;

309   challengedAmount -= _collateralAmount;

330   challengedAmount -= _size;

112   cooldown = expiration; // since expiration is immutable, we put it under cooldown until the end

272   cooldown = expiration;

82   limit = _limit;

99   limit -= reduction + _minimum;

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol


```solidity
file:/contracts/Equity.sol
146  totalVotesAtAnchor = uint192(totalVotes() - roundingLoss - lostVotes);

147  totalVotesAnchorTime = anchorTime();

221   delegates[msg.sender] = delegate;

161  voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); // new example anchor is only 21 / 11 = 1 block in the past

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol


```solidity
file:/contracts/Frankencoin.sol
169    minterReserveE6 += _amount * _reservePPM; // minter reserve must be kept accurately in order to ensure we can get back to exactly 0

196 minterReserveE6 -= amount * reservePPM;

227     minterReserveE6 -= targetTotalBurnAmount * _reservePPM; // reduce reserve requirements by original ratio

253    minterReserveE6 -= freedAmount * _reservePPM; // reduce reserve requirements by original ratio

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol


```solidity
file:
156   _balances[sender] -= amount;

157   _balances[recipient] += amount;

185   _balances[recipient] += amount;

204 _balances[account] -= amount

222  _allowances[owner][spender] = value;

184   _totalSupply += amount;

203   _totalSupply -= amount;

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol


## [G-10] USE ASSEMBLY TO CHECK FOR ADDRESS(0) 


```solidity
file:/contracts/ERC20.sol
152   require(recipient != address(0));

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L152



```solidity
file:/contracts/ERC20.sol
180   require(recipient != address(0));

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L180



```solidity
file:contracts/ERC20PermitLight.sol
56      require(recoveredAddress != address(0) && recoveredAddress == owner, "INVALID_SIGNER");

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L56





## [G-11] Use nested if and, avoid multiple check 

### Details 

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

```solidity
file:/contracts/Position.sol
294   if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L294


```solidity
file:/contracts/Frankencoin.sol
84  if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();

85  if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84-L85


```solidity
file:/contracts/Frankencoin.sol
267    if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L267

