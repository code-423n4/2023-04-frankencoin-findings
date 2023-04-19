# Report


## Low Issues


| |Issue|
|-|:-|
| [L-1](#L-1) | Misleading liquidation price documentation |
| [L-2](#L-2) | Unsafe cast may overflow |
| [L-3](#L-3) | `cooldown` can be less than advertised |
| [L-4](#L-4) | `block.number` is not reliable |
| [L-5](#L-5) | Adjusting a position does not require to wait for cooldown |
| [L-6](#L-6) | Wrong comment in `notifyChallengeSucceeded()` |
| [L-7](#L-7) | Denominator can be zero |
| [L-8](#L-8) | Lack of validation in `splitChallenge()` |
| [L-9](#L-9) | DOS vector in `StablecoinBridge.mint()` |
| [L-10](#L-10) | Avoid multiplication after division |
| [L-11](#L-11) | `_burn()` should check `account` is not the zero address |

### L-1 Misleading liquidation price documentation 
As per the documentation:
```
Anyone can challenge a position if they believe that the liquidation price is below the true value of the collateral
```

But looking at the invariant in `Position` it is the opposite: a position should be challenged if its price is **above** the true value of collateral.

e.g:

if collateral is `USDT` and `price == 2e18`, the position should be challenged (say the position is collateralized with `1M USDT` and minted `2M ZCHF`, it is undercollateralized if we look at the current CHF/USD pair)

```solidity
File: Position.sol
282:     function checkCollateral(uint256 collateralReserve, uint256 atPrice) internal view {
283:         if (collateralReserve * atPrice < minted * ONE_DEC18) revert InsufficientCollateral();
284:     }
```



### L-2 Unsafe cast may overflow
Solidity handles overflows for basic math operations but not for casting. Unsafe casting may lead to truncation if `XXargument` is greater than `type(uintXX).max`.

It is very unlikely to happen in the protocol - for votes with `uint192`, `block.number` with `uint64` (even with the shift) or `PPM` and `uint32`, but consider using a [safeCast library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeCast.sol).

```solidity
File: Equity.sol

146:         totalVotesAtAnchor = uint192(totalVotes() - roundingLoss - lostVotes);

161:             voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); // new example anchor is only 21 / 11 = 1 block in the past

173:         return uint64(block.number << BLOCK_TIME_RESOLUTION_BITS);

```

```solidity
File: Position.sol

187:             return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));

```




### L-3 `cooldown` can be less than advertised
[The documentation](https://docs.frankencoin.com/positions/open) specifies that:

```
Minting is not possible until the initialization period of seven days has passed
```

This is also echoed in the constructor of `Position`:

```solidity
Position.sol
64: start = block.timestamp + initPeriod; // one week time to deny the position
65: cooldown = start;
```

But in reality, it is possible for `initPeriod` to be as low as 3 days

```
Position.sol
53: require(initPeriod >= 3 days);
```

This can be dangerous in certain cases:
For example, imagine a malicious `Position` is created on a Thursday evening, and valid `FPS` holders do not pay attention to it until the following Monday, as they believe it is not possible to mint before a 7 day period.

Indicate clearly in documentation that a new position can mint `ZCHF` after 3 days.



### L-4 `block.number` is not reliable
`block.number` is not a fully accurate way to measure time, as [there can be empty slots](https://ethereum.org/en/developers/docs/blocks/#block-time), which would make the time between two conscutive blocks greater than expected. Use `block.timestamp` to measure passage of time.

```solidity
File: Equity.sol

173:         return uint64(block.number << BLOCK_TIME_RESOLUTION_BITS);

```

### L-5 Adjusting a position does not require to wait for cooldown
The [documentation](https://docs.frankencoin.com/positions/adjust) states that:

```
Once you are the proud owner of a position whose cooldown period has passed, you can start to adjust it
```

But there is no `noCooldown` modifier on `adjust()`:

```solidity
Position.sol
132:     function adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) public onlyOwner {
133:         if (newPrice != price){
134:             adjustPrice(newPrice);
135:         }
```

This means it is possible to increase the price, add collateral and lower the minted amount before the cooldown.

Add a no `noCooldown` modifier to `adjust()`

### L-6 Wrong comment in `notifyChallengeSucceeded()`

`_size` has the same decimals as the collateral token, not 18.

```solidity
File: Position.sol
326:  * @param _size     size of the collateral bid for (dec 18)
```

### L-7 Denominator can be zero
It is technically possible to launch a challenge with `challenge.size == 0` - in the edge case where the position in question has `collateralBalance == 0` (meaning the [check](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L294) in `Position.notifyChallengeStarted()` would pass).

This would make the following line in `splitChallenge` revert 



```solidity
File: MintingHub.sol

165:             (challenge.bid * splitOffAmount) / challenge.size
             );

```

There is already a check that the challenge size is greater than the minimum collateral of the position after the split.
For better error handling, add a check to ensure the challenge to split has a sufficient size before the split:

```diff
File: MintingHub.sol
+            require(challenge.size >= min);
159: Challenge memory copy = Challenge(
160:             challenge.challenger,
161:             challenge.position,
162:             splitOffAmount,
163:             challenge.end,
164:             challenge.bidder,
165:             (challenge.bid * splitOffAmount) / challenge.size
166:         );
167:         challenge.bid -= copy.bid;
168:         challenge.size -= copy.size;
169: 
170:         uint256 min = IPosition(challenge.position).minimumCollateral();
171:         require(challenge.size >= min);
```

### L-8 Lack of validation in `splitChallenge()`

There is no check on `splitOffAmount`.
If it is set to a value greater than `challenge.size`, it will make the following line revert with an underflow error:

```solidity
File: MintingHub.sol
167:         challenge.bid -= copy.bid;
```

This can happen if two calls to `splitChallenge` with the same challenge are made in the same block, and the second call processed is trying to split using an amount greater than the challenge size after the first split.

For better error handling, add a check to ensure the split amount is less than the current size:

```diff
File: MintingHub.sol
+            require(challenge.size >= splitOffAmount, "split amount greater than size");
159: Challenge memory copy = Challenge(
160:             challenge.challenger,
161:             challenge.position,
162:             splitOffAmount,
163:             challenge.end,
164:             challenge.bidder,
165:             (challenge.bid * splitOffAmount) / challenge.size
166:         );
167:         challenge.bid -= copy.bid;
168:         challenge.size -= copy.size;
169: 
170:         uint256 min = IPosition(challenge.position).minimumCollateral();
171:         require(challenge.size >= min);
```

### L-9 DOS vector in `StablecoinBridge.mint()`

The bridge only allows minting of `ZCHF` if its current balance of `XCHF` does not exceed the `limit`

```solidity
File: StablecoinBridge.sol
49:     function mintInternal(address target, uint256 amount) internal {
50:         require(block.timestamp <= horizon, "expired");
51:         require(chf.balanceOf(address(this)) <= limit, "limit");
```

This brings a DOS vector where an attacker can DOS minting calls by front-running them with a mint() call, hitting the `limit` so that the victim's call reverts.

As per the tests scripts, the `limit` would be set to `10M`, meaning the attack is currently impossible as the `XCHF` has a supply of ~`3M` tokens.

However, this supply is not harcoded and the owner of `XCHF` (a Gnosis Wallet) can technically [mint as many tokens as they want](https://etherscan.io/token/0xB4272071eCAdd69d933AdcD19cA99fe80664fc08#code#L460).


As a bridge is supposed to be live for a one year period, we suggest revisiting the `limit` variable to make it modifiable by trusted entities in case the `XCHF` supply were to change.
You could also simply compute `limit` as a value proportional to `XCHF.totalSupply()` using a view function, but that would make minting calls much more expensive due to gas costs.


### L-10 Avoid multiplication after division
This can result in unnecessary precision loss

```solidity
File: Frankencoin.sol
204: function calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) public view returns (uint256) {
205:       uint256 theoreticalReserve = _reservePPM * mintedAmount / 1000000;
206:       uint256 currentReserve = balanceOf(address(reserve));
207:       if (currentReserve < minterReserve()){
208:          // not enough reserves, owner has to take a loss
209:          return theoreticalReserve * currentReserve / minterReserve();//@audit precision loss
```

If  `_reservePPM * mintedAmount` does not have enough trailing zeros, it will get truncated by `1000000`, resulting in precision loss in `theoreticalReserve * currentReserve / minterReserve()`.

Change to: 

```diff
File: Frankencoin.sol
204: function calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) public view returns (uint256) {
-205:       uint256 theoreticalReserve = _reservePPM * mintedAmount / 1000000;
206:       uint256 currentReserve = balanceOf(address(reserve));
207:       if (currentReserve < minterReserve()){
208:          // not enough reserves, owner has to take a loss
-209:          return theoreticalReserve * currentReserve / minterReserve();
+209:          return _reservePPM * mintedAmount * currentReserve / minterReserve() / 100000;
```

### L-11 `_burn()` should check `account` is not the zero address
As per the function comments:


```solidity
File: ERC20.sol
195:      * Requirements
196:      *
197:      * - `account` cannot be the zero address.
```

But there is no such check in the function.

Change to: 

```diff
File: ERC20.sol
200:     function _burn(address account, uint256 amount) internal virtual {
+            require(account != address(0), "address 0");
201:         _beforeTokenTransfer(account, address(0), amount);
202: 
203:         _totalSupply -= amount;
204:         _balances[account] -= amount;
205:         emit Transfer(account, address(0), amount);
206:     }
```

## Non Critical Issues


| |Issue|
|-|:-|
| [NC-1](#NC-1) | Constants on the left are better |
| [NC-2](#NC-2) | `2**128 - 1` should be re-written as `type(uint128).max` |
| [NC-3](#NC-3) | Contract layout not following Solidity standard practice |
| [NC-4](#NC-4) | Order of functions not following Solidity standard practice |
| [NC-5](#NC-5) | Naming conventions of functions |
| [NC-6](#NC-6) | Inconsistent use of `uint` |
| [NC-7](#NC-7) | Assembly blocks should have comments |


### NC-1 Constants on the left are better
This is [safer in case you forget to put a double equals sign](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html), as the compiler wil throw an error instead of assigning the value to the variable.

*Instances (9)*:
```solidity
File: ERC20PermitLight.sol

56:             require(recoveredAddress != address(0) && recoveredAddress == owner, "INVALID_SIGNER");

```

```solidity
File: Equity.sol

145:         uint256 lostVotes = from == address(0x0) ? 0 : (anchorTime() - voteAnchor[from]) * amount;

226:         if (owner == delegate){

228:         } else if (owner == address(0x0)){

242:         require(msg.sender == address(zchf), "caller must be zchf");

```

```solidity
File: MintingHub.sol

259:         address recipient = challenge.bidder == address(0x0) ? msg.sender : challenge.bidder;

```

```solidity
File: Position.sol

250:         if (token == address(collateral)){

```

```solidity
File: StablecoinBridge.sol

76:         if (msg.sender == address(chf)){

78:         } else if (msg.sender == address(zchf)){

```


### NC-2 `2**128 - 1` should be re-written as `type(uint128).max`

*Instances (1)*:
```diff
File: Equity.sol

-253:         require(totalSupply() < 2**128, "total supply exceeded");
+253:         require(totalSupply() <= type(uint128).max, "total supply exceeded");

```



### NC-3 Contract layout not following Solidity standard practice
Follow Solidity's [style guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout) for contract layout: Type declarations, State variables, Events, Modifiers, and Functions

In `Frankencoin`, errors are declared in the middle of the contract. Move them before the constructor.
*Instances (5)*:
```solidity
File: Frankencoin.sol

92: error PeriodTooShort();
93: error FeeTooLow();
94: error AlreadyRegistered();

130: error NotMinter();

159: error TooLate();

```

### NC-4 Order of functions not following Solidity standard practice
Follow Solidity's [style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) for function ordering: constructor, receive/fallback, external, public, internal and private.

*Instances (24)*:
```solidity
File: ERC20.sol

85:     function transfer(address recipient, uint256 amount) public virtual override returns (bool) {

93:     function allowance(address owner, address spender) external view override returns (uint256) {//@audit external defined after public

97:     function allowanceInternal(address owner, address spender) internal view virtual returns (uint256) {

108:     function approve(address spender, uint256 value) external override returns (bool) {//@audit external defined after internal

125:     function transferFrom(address sender, address recipient, uint256 amount) external override returns (bool) {//@audit external defined after internal

151:     function _transfer(address sender, address recipient, uint256 amount) internal virtual {

162:     function transferAndCall(address recipient, uint256 amount, bytes calldata data) external override returns (bool) {//@audit external defined after internal

```


```solidity
File: Equity.sol

112:     function _beforeTokenTransfer(address from, address to, uint256 amount) override internal {

127:     function canRedeem() external view returns (bool){//@audit external defined after internal

135:     function canRedeem(address owner) public view returns (bool) {

144:     function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {

157:     function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){

172:     function anchorTime() internal view returns (uint64){

179:     function votes(address holder) public view returns (uint256) {//@audit public defined after internal

220:     function delegateVoteTo(address delegate) external {

225:     function canVoteFor(address delegate, address owner) internal view returns (bool) {

241:     function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool) {//@audit external defined after internal

266:     function calculateSharesInternal(uint256 capitalBefore, uint256 investment) internal view returns (uint256) {

275:     function redeem(address target, uint256 shares) public returns (uint256) {//@audit public defined after internal

290:     function calculateProceeds(uint256 shares) public view returns (uint256) {//@audit public defined after internal

309:     function restructureCapTable(address[] calldata helpers, address[] calldata addressesToWipe) public {//@audit public defined after internal

```

```solidity
File: Frankencoin.sol

102:    function allowanceInternal(address owner, address spender) internal view override returns (uint256) {//@audit public defined after internal

117:    function minterReserve() public view returns (uint256) {

125:    function registerPosition(address _position) override external {//@audit external defined after public

138:    function equity() public view returns (uint256) {

152:    function denyMinter(address _minter, address[] calldata _helpers, string calldata _message) override external {//@audit external defined after public

204:    function calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) public view returns (uint256) {

223:    function burnFrom(address payer, uint256 targetTotalBurnAmount, uint32 _reservePPM) external override minterOnly returns (uint256) {//@audit external defined after public

235:    function calculateFreedAmount(uint256 amountExcludingReserve /* 41 */, uint32 reservePPM /* 20% */) public view returns (uint256){

251:    function burnWithReserve(uint256 _amountExcludingReserve, uint32 _reservePPM) external override minterOnly returns (uint256) {//@audit external defined after public

```


```solidity
File: MintingHub.sol

124:     function clonePosition(address position, uint256 _initialCollateral, uint256 _initialMint) public validPos(position) returns (address) {

140:     function launchChallenge(address _positionAddr, uint256 _collateralAmount) external validPos(_positionAddr) returns (uint256) {//@audit external defined after public

188:     function minBid(Challenge storage challenge) internal view returns (uint256) {

199:     function bid(uint256 _challengeNumber, uint256 _bidAmountZCHF, uint256 expectedSize) external {//@audit external defined after internal

252:     function end(uint256 _challengeNumber, bool postponeCollateralReturn) public {

281:     function returnPostponedCollateral(address collateral, address target) external {//@audit external defined after public

287:     function returnCollateral(Challenge storage challenge, bool postpone) internal {

314:     function clonePosition(address _existing) external returns (address);//@audit external defined after internal

```


```solidity
File: Position.sol

169:     function collateralBalance() internal view returns (uint256){

177:     function mint(address target, uint256 amount) public onlyOwner noChallenge noCooldown alive {//@audit public defined after internal

202:     function restrictMinting(uint256 period) internal {

227:     function repay(uint256 amount) public onlyOwner {//@audit public defined after internal

240:     function notifyRepaidInternal(uint256 amount) internal {

249:     function withdraw(address token, address target, uint256 amount) external onlyOwner {//@audit external defined after internal

286:     function emitUpdate() internal {

292:     function notifyChallengeStarted(uint256 size) external onlyHub {//@audit external defined after internal


```


```solidity
File: StablecoinBridge.sol

49:     function mintInternal(address target, uint256 amount) internal {

55:     function burn(uint256 amount) external {//@audit external defined after internal


```

### NC-5 Naming conventions of functions
follow Solidity standard naming conventions: internal and private functions should start with an underscore

*Instances (7)*:


```solidity
File: Equity.sol

157:     function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){

172:     function anchorTime() internal view returns (uint64){

```


```solidity
File: Ownable.sol


45:     function requireOwner(address sender) internal view {

```

```solidity
File: Position.sol

202:     function restrictMinting(uint256 period) internal {

240:     function notifyRepaidInternal(uint256 amount) internal {

282:     function checkCollateral(uint256 collateralReserve, uint256 atPrice) internal view {

286:     function emitUpdate() internal {

```


### NC-6 Inconsistent use of `uint`
Throughout the contracts, `uint256` is used, but `uint` is used in some instances. For consistency, use only one or the other. This can also be a problem for [structured data signing](https://eips.ethereum.org/EIPS/eip-712#definition-of-typed-structured-data-%F0%9D%95%8A)

*Instances (4)*:
```solidity
File: Equity.sol

91:     event Trade(address who, int amount, uint totPrice, uint newprice); // amount pos or neg for mint or redemption

91:     event Trade(address who, int amount, uint totPrice, uint newprice); // amount pos or neg for mint or redemption

192:         for (uint i=0; i<helpers.length; i++){

196:             for (uint j=i+1; j<helpers.length; j++){

```


### NC-7 Assembly blocks should have comments
Comments clearly explaining what each assembly instruction does will make it easier for users to review the code.

*Instances (1)*:
```solidity
File: PositionFactory.sol

39:         assembly {

```
