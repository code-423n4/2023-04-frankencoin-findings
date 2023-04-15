
## Summary

### Gas Optimizations
| |Issue|Instances\*|Gas Saved\*\*|
|-|:-|:-:|:-:|
| [G&#x2011;01] | Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, where appropriate | 1 |  10,500 |
| [G&#x2011;02] | Avoid contract existence checks by using low level calls | 38 |  At least 3800 |
| [G&#x2011;03] | Functions guaranteed to revert when called by normal users can be marked `payable` | 14 |  At least 294 |
| [G&#x2011;04] | Multiple accesses of a mapping should use a local variable cache | 8 |  At least 336 |
| [G&#x2011;05] | Multiple accesses of an array should use a local variable cache | 2 |  - |
| [G&#x2011;06] | State variables should be cached in stack variables rather than re-reading them from storage | 34 |  At least 3,400 |
| [G&#x2011;07] | Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if`-statement | 16 |  At least1360 |
| [G&#x2011;08] | The result of internal view function calls should be cached rather than re-calling the function | 1 |  - |

Total: 114 instances over 8 issues with **at least 19,690 gas** saved.

\* Exluding any instance from the automated findings (in case the automated findings don't contain all instances).

\*\* Gas totals use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. Gas savings are calculated as the sum of all total savings of each public/external function (including any internal/external function call made by this function directly or indirectly) available for the end user. "At least" is written were I didn't calculated the savings of all the instances of an issue on each external call. For those instances which weren't taken into consideration, I calculated the raw savings of the instance - practically assuming that the ineffient code from that instance would be executed (directly or indirectly) on at least one public/external function available for the end user (and isn't dead code).

## Gas Optimizations

### [G&#x2011;01]  Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, where appropriate
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (**20000 gas**) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save **~42 gas per access** due to [not having to recalculate the key's keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation's associated stack operations.

*There is 1 instance of this issue:*

[contracts\Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L83-L88)
```solidity
/// @audit can be a mapping of an address to a struct of address and uint64
83      mapping (address => address) public delegates;
88      mapping (address => uint64) private voteAnchor; // 40 Bit for the block number, 24 Bit sub-block time resolution
```

- Saves 1 Gcoldsload (**2100 gas**) on [`Equity.votes(address sender, address[] calldata helpers)`](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L195) (see [`Equity.canVoteFor(address delegate, address owner)`](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L231)) and [`Equity.votes(address sender, address[] calldata helpers)`](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L199) (see [`Equity.votes(address holder)`](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L180))
- Saves 1 Gcoldsload (**2100 gas**) on [`Equity.checkQualified(address sender, address[] calldata helpers)`](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L210)
- Saves 1 Gcoldsload (**2100 gas**) on [`Equity.restructureCapTable(address[] calldata helpers, address[] calldata addressesToWipe)`](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L311)
- Saves 1 Gcoldsload (**2100 gas**) on [`Frankencoin.denyMinter(address _minter, address[] calldata _helpers, string calldata _message)`](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L154)
- Saves 1 Gcoldsload (**2100 gas**) on [`Position.deny(address[] calldata helpers, string calldata message)`](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L109)

### [G&#x2011;02]  Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including `EXTCODESIZE` (**100 gas**), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

*There are 38 instances of this issue:*

[contracts\Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L109)
```solidity
109         return VALUATION_FACTOR * zchf.equity() * ONE_DEC18 / totalSupply();
```

[contracts\Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L243)
```solidity
243         uint256 equity = zchf.equity();
```

[contracts\Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L263)
```solidity
263         return calculateSharesInternal(zchf.equity(), investment);
```

[contracts\Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L279)
```solidity
279         zchf.transfer(target, proceeds);
```

[contracts\Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L292)
```solidity
292         uint256 capital = zchf.equity();
```

[contracts\Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L310)
```solidity
310         require(zchf.equity() < MINIMUM_EQUITY);
```

[contracts\ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L165)
```solidity
165             success = IERC677Receiver(recipient).onTokenTransfer(msg.sender, amount, data);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L93-L105)
```solidity
93              POSITION_FACTORY.createNewPosition(
94                  msg.sender,
95                  address(zchf),
96                  _collateralAddress,
97                  _minCollateral,
98                  _mintingMaximum,
99                  _initPeriodSeconds,
100                 _expirationSeconds,
101                 _challengeSeconds,
102                 _mintingFeePPM,
103                 _liqPrice,
104                 _reservePPM
105             )
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L108)
```solidity
108         zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L110)
```solidity
110         IERC20(_collateralAddress).transferFrom(msg.sender, address(pos), _initialCollateral);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L116)
```solidity
116         require(zchf.isPosition(position) == address(this), "not our pos");
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L126)
```solidity
126         uint256 limit = existing.reduceLimitForClone(_initialMint);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L127)
```solidity
127         address pos = POSITION_FACTORY.clonePosition(position);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L129)
```solidity
129         existing.collateral().transferFrom(msg.sender, address(pos), _initialCollateral);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L129)
```solidity
129         existing.collateral().transferFrom(msg.sender, address(pos), _initialCollateral);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L142)
```solidity
142         IERC20(position.collateral()).transferFrom(msg.sender, address(this), _collateralAmount);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L142)
```solidity
142         IERC20(position.collateral()).transferFrom(msg.sender, address(this), _collateralAmount);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L144)
```solidity
144         challenges.push(Challenge(msg.sender, position, _collateralAmount, block.timestamp + position.challengePeriod(), address(0x0), 0));
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L170)
```solidity
170         uint256 min = IPosition(challenge.position).minimumCollateral();
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L204)
```solidity
204             zchf.transfer(challenge.bidder, challenge.bid); // return old bid
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L210)
```solidity
210             zchf.transferFrom(msg.sender, challenge.challenger, _bidAmountZCHF);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L225)
```solidity
225             zchf.transferFrom(msg.sender, address(this), _bidAmountZCHF);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L263)
```solidity
263             IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L268)
```solidity
268             zchf.transfer(owner, effectiveBid - fundsNeeded);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L272)
```solidity
272         zchf.transfer(challenge.challenger, reward); // pay out the challenger reward
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L284)
```solidity
284         IERC20(collateral).transfer(target, amount);
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L111)
```solidity
111         IReserve(zchf.reserve()).checkQualified(msg.sender, helpers);
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L138)
```solidity
138             collateral.transferFrom(msg.sender, address(this), newCollateral - colbal);
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L142)
```solidity
142             zchf.burnFrom(msg.sender, minted - newMinted, reserveContribution);
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L170)
```solidity
170         return IERC20(collateral).balanceOf(address(this));
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L228)
```solidity
228         IERC20(zchf).transferFrom(msg.sender, address(this), amount);
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L233)
```solidity
233         uint256 actuallyBurned = IFrankencoin(zchf).burnWithReserve(burnable, reserveContribution);
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L253)
```solidity
253             IERC20(token).transfer(target, amount);
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L269)
```solidity
269         IERC20(collateral).transfer(target, amount);
```

[contracts\PositionFactory.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L32)
```solidity
32          Position clone = Position(createClone(existing.original()));
```

[contracts\StablecoinBridge.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L45)
```solidity
45          chf.transferFrom(msg.sender, address(this), amount);
```

[contracts\StablecoinBridge.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L51)
```solidity
51          require(chf.balanceOf(address(this)) <= limit, "limit");
```

[contracts\StablecoinBridge.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L69)
```solidity
69          chf.transfer(target, amount);
```

### [G&#x2011;03]  Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are 
`CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost.

*There are 14 instances of this issue:*

[contracts\Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L241-L242)
```solidity
241     function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool) {
242         require(msg.sender == address(zchf), "caller must be zchf");
```

[contracts\Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L172)
```solidity
172    function mint(address _target, uint256 _amount) override external minterOnly {
```

[contracts\Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L262)
```solidity
262    function burn(address _owner, uint256 _amount) override external minterOnly {
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L76)
```solidity
76      function initializeClone(address owner, uint256 _price, uint256 _limit, uint256 _coll, uint256 _mint) external onlyHub {
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L97)
```solidity
97      function reduceLimitForClone(uint256 _minimum) external noChallenge noCooldown alive onlyHub returns (uint256) {
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L132)
```solidity
132     function adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) public onlyOwner {
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L159)
```solidity
159     function adjustPrice(uint256 newPrice) public onlyOwner noChallenge {
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L177)
```solidity
177     function mint(address target, uint256 amount) public onlyOwner noChallenge noCooldown alive {
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L227)
```solidity
227     function repay(uint256 amount) public onlyOwner {
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L249)
```solidity
249     function withdraw(address token, address target, uint256 amount) external onlyOwner {
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L263)
```solidity
263     function withdrawCollateral(address target, uint256 amount) public onlyOwner noChallenge noCooldown {
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L292)
```solidity
292     function notifyChallengeStarted(uint256 size) external onlyHub {
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L304)
```solidity
304     function tryAvertChallenge(uint256 _collateralAmount, uint256 _bidAmountZCHF) external onlyHub returns (bool) {
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L329)
```solidity
329     function notifyChallengeSucceeded(address _bidder, uint256 _bid, uint256 _size) external onlyHub returns (address, uint256, uint256, uint256, uint32) {
```

### [G&#x2011;04]  Multiple accesses of a mapping should use a local variable cache
The instances below point to the second+ access of a value inside a mapping, within a function. Caching a mapping's value in a local `storage` or `calldata` variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves **~42 gas per access** due to not having to recalculate the key's keccak256 hash (Gkeccak256 - **30 gas**) and that calculation's associated stack operations.

*There are 8 instances of this issue:*

[contracts\ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L155)
```solidity
/// @audit _balances[sender] on line 155
155         if (_balances[sender] < amount) revert ERC20InsufficientBalance(sender, _balances[sender], amount);
```

[contracts\ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L156)
```solidity
/// @audit _balances[sender] on line 155
156         _balances[sender] -= amount;
```

[contracts\Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L88)
```solidity
/// @audit minters[_minter] on line 86
88        minters[_minter] = block.timestamp + _applicationPeriod;
```

[contracts\Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L155)
```solidity
/// @audit minters[_minter] on line 153
155       delete minters[_minter];
```

[contracts\Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L294)
```solidity
/// @audit minters[_minter] on line 294
294       return minters[_minter] != 0 && block.timestamp >= minters[_minter];
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L283)
```solidity
/// @audit pendingReturns[collateral] on line 282
283         delete pendingReturns[collateral][msg.sender];
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L283)
```solidity
/// @audit pendingReturns[collateral][msg.sender] on line 282
283         delete pendingReturns[collateral][msg.sender];
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L283)
```solidity
/// @audit pendingReturns[collateral] on line 282
283         delete pendingReturns[collateral][msg.sender];
```

### [G&#x2011;05]  Multiple accesses of an array should use a local variable cache
The instances below point to the second+ access of a value inside an array, within a function. Caching an array's element avoids recalculating the array offsets into memory/calldata and avoids the boundaries check.

*There are 2 instances of this issue:*

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L213)
```solidity
/// @audit challenges[_challengeNumber] on line 200
213             delete challenges[_challengeNumber];
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L275)
```solidity
/// @audit challenges[_challengeNumber] on line 253
275         delete challenges[_challengeNumber];
```


### [G&#x2011;06]  State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (**100 gas**) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*There are 34 instances of this issue:*

[contracts\ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L155)
```solidity
/// @audit _balances[sender] on line 155
155         if (_balances[sender] < amount) revert ERC20InsufficientBalance(sender, _balances[sender], amount);
```

[contracts\ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L156)
```solidity
/// @audit _balances[sender] on line 155
156         _balances[sender] -= amount;
```

[contracts\Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L294)
```solidity
/// @audit minters[_minter] on line 294
294       return minters[_minter] != 0 && block.timestamp >= minters[_minter];
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L160)
```solidity
/// @audit challenge.challenger on line 158
160             challenge.challenger,
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L167)
```solidity
/// @audit challenge.bid on line 165
167         challenge.bid -= copy.bid;
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L168)
```solidity
/// @audit challenge.size on line 165
168         challenge.size -= copy.size;
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L170)
```solidity
/// @audit challenge.position on line 161
170         uint256 min = IPosition(challenge.position).minimumCollateral();
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L171)
```solidity
/// @audit challenge.size on line 168
171         require(challenge.size >= min);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L176)
```solidity
/// @audit challenge.challenger on line 158
176         emit ChallengeStarted(challenge.challenger, address(challenge.position), challenge.size, _challengeNumber);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L176)
```solidity
/// @audit challenge.position on line 161
176         emit ChallengeStarted(challenge.challenger, address(challenge.position), challenge.size, _challengeNumber);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L176)
```solidity
/// @audit challenge.size on line 168
176         emit ChallengeStarted(challenge.challenger, address(challenge.position), challenge.size, _challengeNumber);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L204)
```solidity
/// @audit challenge.bid on line 203
204             zchf.transfer(challenge.bidder, challenge.bid); // return old bid
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L208)
```solidity
/// @audit challenge.size on line 202
208         if (challenge.position.tryAvertChallenge(challenge.size, _bidAmountZCHF)) {
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L211)
```solidity
/// @audit challenge.position on line 208
211             challenge.position.collateral().transfer(msg.sender, challenge.size);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L211)
```solidity
/// @audit challenge.size on line 202
211             challenge.position.collateral().transfer(msg.sender, challenge.size);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L212)
```solidity
/// @audit challenge.position on line 208
212             emit ChallengeAverted(address(challenge.position), _challengeNumber);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L218)
```solidity
/// @audit challenge.end on line 201
218             if (earliestEnd >= challenge.end) {
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L259)
```solidity
/// @audit challenge.bidder on line 259
259         address recipient = challenge.bidder == address(0x0) ? msg.sender : challenge.bidder;
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L261)
```solidity
/// @audit challenge.bid on line 260
261         if (effectiveBid < challenge.bid) {
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L263)
```solidity
/// @audit challenge.bidder on line 259
263             IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L263)
```solidity
/// @audit challenge.bid on line 260
263             IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L272)
```solidity
/// @audit challenge.challenger on line 254
272         zchf.transfer(challenge.challenger, reward); // pay out the challenger reward
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L273)
```solidity
/// @audit challenge.position on line 260
273         emit ChallengeSucceeded(address(challenge.position), challenge.bid, _challengeNumber);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L273)
```solidity
/// @audit challenge.bid on line 260
273         emit ChallengeSucceeded(address(challenge.position), challenge.bid, _challengeNumber);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L292)
```solidity
/// @audit challenge.challenger on line 291
292             emit PostPonedReturn(collateral, challenge.challenger, challenge.size);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L292)
```solidity
/// @audit challenge.size on line 291
292             emit PostPonedReturn(collateral, challenge.challenger, challenge.size);
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L81)
```solidity
/// @audit price on line 80
81          if (price > _price) revert InsufficientCollateral();
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L99)
```solidity
/// @audit limit on line 98
99          limit -= reduction + _minimum;
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L142)
```solidity
/// @audit minted on line 141
142             zchf.burnFrom(msg.sender, minted - newMinted, reserveContribution);
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L150)
```solidity
/// @audit minted on line 149
150             mint(msg.sender, newMinted - minted);
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L196)
```solidity
/// @audit minted on line 194
196         minted += amount;
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L241)
```solidity
/// @audit minted on line 241
241         if (amount > minted) revert RepaidTooMuch(amount - minted);
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L242)
```solidity
/// @audit minted on line 241
242         minted -= amount;
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L349)
```solidity
/// @audit minted on line 349
349         uint256 repayment = minted < volumeZCHF ? minted : volumeZCHF; // how much must be burned to make things even
```

### [G&#x2011;07]  Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if`-statement
`require(a <= b); x = b - a` => `require(a <= b); unchecked { x = b - a }`.

*There are 16 instances of this issue:*

[contracts\Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L247)
```solidity
/// @audit equity >= amount because of the ternary condition on line 247
247         uint256 shares = equity <= amount ? 1000 * ONE_DEC18 : calculateSharesInternal(equity - amount, amount);
```

[contracts\ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L132)
```solidity
/// @audit currentAllowance >= amount because of the if condition on line 131
132             _approve(sender, msg.sender, currentAllowance - amount);
```

[contracts\Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L144)
```solidity
/// @audit balance >= minReserve because of the if condition on line 141
144         return balance - minReserve;
```

[contracts\Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L168)
```solidity
/// @audit _amount >= usableMint because of the calculation of usableMint on line 166
168       _mint(address(reserve), _amount - usableMint); // rest goes to equity as reserves or as fees
```

[contracts\Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L286)
```solidity
/// @audit _amount >= reserveLeft because of the if condition on line 282
286          _mint(msg.sender, _amount - reserveLeft);
```

[contracts\MathUtil.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MathUtil.sol#L26)
```solidity
/// @audit xOld >= x because of the ternary condition on line 26
26              cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
```

[contracts\MathUtil.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MathUtil.sol#L26)
```solidity
/// @audit x >= xOld because of the ternary condition on line 26
26              cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L263)
```solidity
/// @audit challenge.bid >= effectiveBid because of the if condition on line 261
263             IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L268)
```solidity
/// @audit effectiveBid >= fundsNeeded because of the if condition on line 267
268             zchf.transfer(owner, effectiveBid - fundsNeeded);
```

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L270)
```solidity
/// @audit fundsNeeded >= effectiveBid because of the if condition on line 269
270             zchf.notifyLoss(fundsNeeded - effectiveBid); // ensure we have enough to pay everything
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L138)
```solidity
/// @audit newCollateral >= colbal because of the if condition on line 137
138             collateral.transferFrom(msg.sender, address(this), newCollateral - colbal);
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L142)
```solidity
/// @audit minted >= newMinted because of the if condition on line 141
142             zchf.burnFrom(msg.sender, minted - newMinted, reserveContribution);
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L146)
```solidity
/// @audit colbal >= newCollateral because of the if condition on line 145
146             withdrawCollateral(msg.sender, colbal - newCollateral);
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L138)
```solidity
/// @audit newMinted >= minted because of the if condition on line 149
150             mint(msg.sender, newMinted - minted);
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L241)
```solidity
/// @audit amount >= minted because of the if condition on line 241
241         if (amount > minted) revert RepaidTooMuch(amount - minted);
```

[contracts\Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L242)
```solidity
/// @audit minted >= amount because of the if condition on line 241
242         minted -= amount;
```

### [G&#x2011;08]  The result of internal function calls should be cached rather than re-calling the function
The instances below point to the second+ call of the function within a single function.

*There is 1 instance of this issue:*

[contracts\MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L216)
```solidity
/// @audit minBid(challenge) on line 216
216             if (_bidAmountZCHF < minBid(challenge)) revert BidTooLow(_bidAmountZCHF, minBid(challenge));
```
