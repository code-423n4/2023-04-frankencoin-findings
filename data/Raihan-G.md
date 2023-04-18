# Gas Optimizations
#### Note:  The first 4 types is Miss from bots
# SUMMARY 
|      |  Issue    | Instence|
|------|-----------|---------|
|[G‑01]| Functions guaranteed to revert when called by normal users can be marked payable | 13
|[G-02]| Use hardcoded address instead address(this) | 2
|[G‑03]| Use Custom Errors Rather Than revert()/require() Strings To Save Gas | 8
|[G-04]| Public Functions To External | 2
|[G-05]| Using calldata instead of memory for read-only arguments in external functions saves gas | 1
|[G-06]| >= costs less gas than > | 16
|[G-07]|Amounts should be checked for 0 before calling a transfer | 8
|[G-08]| Using bools for storage incurs overhead | 17
|[G-09]|Avoid using state variable in emit | 3
|[G-10]|Change public state variable visibility to private | 1
|[G-11]| Make 3 event parameters indexed when possible | 11
|[G-12]|use Mappings Instead of Arrays | 1
|[G‑13]| Multiple accesses of a mapping/array should use a local variable cache | 12
|[G‑14]| State variables should be cached in stack variables rather than re-reading them from storage | 26
|[G-15]|Use != 0 instead of > 0 for unsigned integer comparison | 6
|[G-16]| The result of a function call should be cached rather than re-calling the function | 6
|[G‑17]| Use nested if statements instead of && | 4
 



## [G‑01] Functions guaranteed to revert when called by normal users can be marked payable
### Not: this miss from bots

```solidity
File: /contracts/Frankencoin.sol
172        function mint(address _target, uint256 _amount) override external minterOnly {

262     function burn(address _owner, uint256 _amount) override external minterOnly {    
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

```solidity
File: /contracts/Position.sol
132    function adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) public onlyOwner {

159     function adjustPrice(uint256 newPrice) public onlyOwner noChallenge {    

177     function mint(address target, uint256 amount) public onlyOwner noChallenge noCooldown alive {   

227     function repay(uint256 amount) public onlyOwner {     

249     function withdraw(address token, address target, uint256 amount) external onlyOwner {

263      function withdrawCollateral(address target, uint256 amount) public onlyOwner noChallenge noCooldown { 

76     function initializeClone(address owner, uint256 _price, uint256 _limit, uint256 _coll, uint256 _mint) external onlyHub { 
97     function reduceLimitForClone(uint256 _minimum) external noChallenge noCooldown alive onlyHub returns (uint256) {  

292     function notifyChallengeStarted(uint256 size) external onlyHub {

304     function tryAvertChallenge(uint256 _collateralAmount, uint256 _bidAmountZCHF) external onlyHub returns (bool) {

329     function notifyChallengeSucceeded(address _bidder, uint256 _bid, uint256 _size) external onlyHub returns (address, uint256, uint256, uint256, uint32) {    
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

## [G-02] Use hardcoded address instead address(this)
### note: this miss from bots

```solidity
116     require(zchf.isPosition(position) == address(this), "not our pos");
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L116


```solidity
File: /contracts/MintingHub.sol
55     original = address(this);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L55


## [G‑03] Use Custom Errors Rather Than revert()/require() Strings To Save Gas
### note this is miss from bots
```solidity
File: /contracts/Equity.sol
242      require(msg.sender == address(zchf), "caller must be zchf");

244      require(equity >= MINIMUM_EQUITY, "insuf equity"); // ensures that the initial deposit is at least 1000 ZCHF

253      require(totalSupply() < 2**128, "total supply exceeded");
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

```solidity
File: /contracts/ERC20PermitLight.sol
30      require(deadline >= block.timestamp, "PERMIT_DEADLINE_EXPIRED");
```
https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/ERC20PermitLight.sol#L30
```solidity
File: /contracts/MintingHub.sol
109     require(_initialCollateral >= _minCollateral, "must start with min col");

116     require(zchf.isPosition(position) == address(this), "not our pos");

```
https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/MintingHub.sol

```solidity
File: /contracts/StablecoinBridge.sol
50      require(block.timestamp <= horizon, "expired");
51      require(chf.balanceOf(address(this)) <= limit, "limit");
```
https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/StablecoinBridge.sol

 ## [G-04] Public Functions To External
The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

### note : miss from bots
```solidity
File: /contracts/Frankencoin.sol
293     function isMinter(address _minter) override public view returns (bool){
    
300     function isPosition(address _position) override public view returns (address){    
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L293#L300




## [G-05] Using calldata instead of memory for read-only arguments in external functions saves gas
When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution.


```solidity
File: /contracts/MintingHub.sol
159     Challenge memory copy = Challenge(
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L159
## [G-06] >= costs less gas than >
The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas

```solidity
File: /contracts/Equity.sol
115     if (amount > 0){
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L115

```solidity
File: /contracts/Frankencoin.sol
84     if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();

85     if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();

104    if (explicit > 0){

153    if (block.timestamp > minters[_minter]) revert TooLate();    
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

```solidity
File: /contracts/MintingHub.sol
203    if (challenge.bid > 0) {
267    if (effectiveBid > fundsNeeded){
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L203-#L267

```solidity
File: /contracts/Position.sol
81     if (price > _price) revert InsufficientCollateral();

137    if (newCollateral > colbal){

160    if (newPrice > price) {

194    if (minted + amount > limit) revert LimitExceeded();

204    if (horizon > cooldown){

241    if (amount > minted) revert RepaidTooMuch(amount - minted);

332    if (_size > colBal){

367    if (block.timestamp > expiration) revert Expired();  

381    if (challengedAmount > 0) revert Challenged();          
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol


## [G-07]Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it's not consistently done in the solution.

```solidity
File: /contracts/Equity.sol
280     zchf.transfer(target, proceeds);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L280

```solidity
File: /contracts/MintingHub.sol
211     challenge.position.collateral().transfer(msg.sender, challenge.size);

272     zchf.transfer(challenge.challenger, reward);

284     IERC20(collateral).transfer(target, amount);

294      challenge.position.collateral().transfer(challenge.challenger, challenge.size); // return the challenger's collateral
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

```solidity
File: /contracts/Position.sol
253     IERC20(token).transfer(target, amount);

269     IERC20(collateral).transfer(target, amount);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

```solidity
File: /contracts/StablecoinBridge.sol
69     chf.transfer(target, amount);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol

## [G-08] Using bools for storage incurs overhead

// Booleans are more expensive than uint256 or any type that takes up a full
// word because each write operation emits an extra SLOAD to first read the
// slot's contents, replace the bits taken up by the boolean, and then write
// back. This is the compiler's defense against contract upgrades and
// pointer aliasing, and it cannot be disabled.

```solidity
File: /contracts/Equity.sol
128      function canRedeem() external view returns (bool){

136      function canRedeem(address owner) public view returns (bool) {

226      function canVoteFor(address delegate, address owner) internal view returns (bool) {

242      function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool) {    
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

```solidity
File: /contracts/ERC20.sol
85      function transfer(address recipient, uint256 amount) public virtual override returns (bool) {

108     function approve(address spender, uint256 value) external override returns (bool) {    

125     function transferFrom(address sender, address recipient, uint256 amount) external override returns (bool) {

162     function transferAndCall(address recipient, uint256 amount, bytes calldata data) external override returns (bool) {         
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol

```solidity
File: /contracts/Frankencoin.sol

293     function isMinter(address _minter) override public view returns (bool){
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#293

```solidity
File: /contracts/MathUtil.sol
21     bool cond;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L21wwz

```solidity
File: /contracts/MintingHub.sol
239     function isChallengeOpen(uint256 _challengeNumber) external view returns (bool) {

252     function end(uint256 _challengeNumber, bool postponeCollateralReturn) public {

287     function returnCollateral(Challenge storage challenge, bool postpone) internal {        
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

```solidity
File: /contracts/Position.sol
120     function getUsableMint(uint256 totalMint, bool afterFees) public view returns (uint256){

304       function tryAvertChallenge(uint256 _collateralAmount, uint256 _bidAmountZCHF) external onlyHub returns (bool) {

360     function isClosed() public view returns (bool) {        
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

```solidity
File: /contracts/StablecoinBridge.sol
75     function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool){
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L75


## [G-09]Avoid using state variable in emit 
 Using a state variable in SetOwner emits wastes gas.

```solidity
File: /contracts/Position.sol
69      emit PositionOpened(_owner, original, _zchf, address(collateral), _liqPrice);

87      emit PositionOpened(owner, original, address(zchf), address(collateral), _price);

287     emit MintingUpdate(collateralBalance(), price, minted, limit);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

## [G-10]Change public state variable visibility to private
 If it is preferred to change the visibility of the owner
 
 ```solidity
 File: /contracts/Ownable.sol
 21      address public owner; 
 ```
 https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L21


## [G-11] Make 3 event parameters indexed when possible
It's the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
File: /contracts/Equity.sol
92     event Trade(address who, int amount, uint totPrice, uint newprice); // amount pos or neg for mint or redemption
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L92

```solidity
File: /contracts/Frankencoin.sol
52    event MinterApplied(address indexed minter, uint256 applicationPeriod, uint256 applicationFee, string message);

53    event MinterDenied(address indexed minter, string message);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

```solidity
File: /contracts/MintingHub.sol
48    event ChallengeStarted(address indexed challenger, address indexed position, uint256 size, uint256 number);

49    event ChallengeAverted(address indexed position, uint256 number);

50    event ChallengeSucceeded(address indexed position, uint256 bid, uint256 number);

51    event NewBid(uint256 challengedId, uint256 bidAmount, address bidder);

52    event PostPonedReturn(address collateral, address indexed beneficiary, uint256 amount);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

```solidity
File: /contracts/Position.sol
41     event PositionOpened(address indexed owner, address original, address zchf, address collateral, uint256 price);

42     event MintingUpdate(uint256 collateral, uint256 price, uint256 minted, uint256 limit);

43     event PositionDenied(address indexed sender, string message); // emitted if closed by governance
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

## [G-12]use Mappings Instead of Arrays
There are two data types to describe lists of data in Solidity, arrays and maps, and their syntax and structure are quite different, allowing each to serve a distinct purpose. While arrays are packable and iterable, mappings are less expensive.

For example, creating an array of cars in Solidity might look like this:



string cars[];
cars = ["ford", "audi", "chevrolet"];

Let’s see how to create a mapping for cars:



mapping(uint => string) public cars

When using the mapping keyword, you will specify the data type for the key (uint) and the value (string). Then you can add some data using the constructor function.



 constructor() public {
        cars[101] = "Ford";
        cars[102] = "Audi";
        cars[103] = "Chevrolet";
    }
}

Except where iteration is required or data types can be packed, it is advised to use mappings to manage lists of data in order to conserve gas. This is beneficial for both memory and storage.

An integer index can be used as a key in a mapping to control an ordered list. Another advantage of mappings is that you can access any value without having to iterate through an array as would otherwise be necessary. 

```solidity
File: /contracts/MintingHub.sol
31     Challenge[] public challenges; // list of open challenges
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L31



## [G‑13] Multiple accesses of a mapping/array should use a local variable cache
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata.

note: Multiple access of this array : _balances[sender]
```solidity
File: /contracts/ERC20.sol
        _beforeTokenTransfer(sender, recipient, amount);
        if (_balances[sender] < amount) revert ERC20InsufficientBalance(sender, _balances[sender], amount);
        _balances[sender] -= amount;
        _balances[recipient] += amount;
        emit Transfer(sender, recipient, amount);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L154-#L158


note: Multiple access of this array : minters[_minter];
```solidity
File: /contracts/Frankencoin.sol
86      if (minters[_minter] != 0) revert AlreadyRegistered();
88      minters[_minter] = block.timestamp + _applicationPeriod;
153     if (block.timestamp > minters[_minter]) revert TooLate();
155     delete minters[_minter];
294     return minters[_minter] != 0 && block.timestamp >= minters[_minter];
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

note: Multiple access of this array : challenges[_challengeNumber]
```solidity
File: /contracts/MintingHub.sol
200     Challenge storage challenge = challenges[_challengeNumber];
213     delete challenges[_challengeNumber];
253     Challenge storage challenge = challenges[_challengeNumber];
275     delete challenges[_challengeNumber];
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol


note: Multiple access of this array : pendingReturns[collateral][msg.sender];
```solidity
File: /contracts/MintingHub.sol
282      uint256 amount = pendingReturns[collateral][msg.sender];
283      delete pendingReturns[collateral][msg.sender];
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L282-#L283

## [G‑14] State variables should be cached in stack variables rather than re-reading them from storage
The instances in this report point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

```solidity
File: /contracts/Position.sol
142     zchf.burnFrom(msg.sender, minted - newMinted, reserveContribution);
143     minted = newMinted;

194     if (minted + amount > limit) revert LimitExceeded();
196     minted += amount;

309     uint256 repayment = minted < volumeZCHF ? minted : volumeZCHF;

204          if (horizon > cooldown){
205          cooldown = horizon;

64        start = block.timestamp + initPeriod; // one week time to deny the position
65        cooldown = start;
66        expiration = start + _duration;

187     return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));

55      original = address(this);
69      emit PositionOpened(_owner, original, _zchf, address(collateral), _liqPrice);

122     return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
124     return totalMint * (1000_000 - reserveContribution) / 1000_000;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol



```solidity
File: /contracts/MintingHub.sol
95      address(zchf),
107     zchf.registerPosition(address(pos));     
108     zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE);

204    zchf.transfer(challenge.bidder, challenge.bid); // return old bid
210    zchf.transferFrom(msg.sender, challenge.challenger, _bidAmountZCHF);
225    zchf.transferFrom(msg.sender, address(this), _bidAmountZCHF);

263    IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);
268    zchf.transfer(owner, effectiveBid - fundsNeeded);
270    zchf.notifyLoss(fundsNeeded - effectiveBid); // ensure we have enough to pay everything
272    zchf.transfer(challenge.challenger, reward); // pay out the challenger reward
273    zchf.burn(repayment, reservePPM); // Repay the challenged part
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol


## [G-15]Use != 0 instead of > 0 for unsigned integer comparison
When dealing with unsigned integer types, comparisons with != 0 are cheaper then with > 0.

```solidity
File: /contracts/Equity.sol
114     if (amount > 0){
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L114

```solidity
File: /contracts/Frankencoin.sol
84      if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();

85      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();

104     if (explicit > 0){
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol


```solidity
File: /contracts/MintingHub.sol
203       if (challenge.bid > 0) {
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L203

```solidity
File: /contracts/Position.sol
381     if (challengedAmount > 0) revert Challenged();
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L381


## [G-16] The result of a function call should be cached rather than re-calling the function

```solidity
File: /contracts/Equity.sol
145     uint256 lostVotes = from == address(0x0) ? 0 : (anchorTime() - voteAnchor[from]) * amount;
147     totalVotesAnchorTime = anchorTime();
```
Mig : first assign function to state var then use in condition

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L145-#L147


```solidity
File: /contracts/Frankencoin.sol
85     if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert    PeriodTooShort();
87     if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();


207    if (currentReserve < `minterReserve()`){
209    return theoreticalReserve * currentReserve / `minterReserve()`;    
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

## [G‑17] Use nested if statements instead of &&

```solidity
File: /contracts/Frankencoin.sol
85     if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();

86     if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();

267     if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

```solidity
File: /contracts/Position.sol
294     if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L294
