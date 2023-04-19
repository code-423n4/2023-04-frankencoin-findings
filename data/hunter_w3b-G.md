## Summary

### Gas Optimizations

|               | Issue                                                                                                                | Instances | Total Gas Saved |
| ------------- | :------------------------------------------------------------------------------------------------------------------- | :-------: | :-------------: |
| [G&#x2011;01] | Avoid `contract existence` checks by using solidity version 0.8.10 or later                                          |     3     |      1300       |
| [G&#x2011;02] | Add unchecked {} for subtractions where the operands can't underflow because of previous require and if statement    |    12     |        -        |
| [G&#x2011;03] | Using fixed bytes is cheaper than using string                                                                       |     4     |        -        |
| [G&#x2011;04] | Multiple address/id mappings can be combined into a `single mapping` of and address/id to a struct where appropriate |     8     |        -        |
| [G&#x2011;05] | State variables should be cached in `stack variables` rather than re-reading them from `storage`                     |    40     |      3880       |
| [G&#x2011;06] | Stack variable used as a cheaper cache for `state variables` is `only used once`                                     |     8     |       24        |
| [G&#x2011;07] | Not Using the named `return` variables when a function returns, `WASTES DEPLOYMENT GAS`                              |     1     |        -        |
| [G&#x2011;08] | Can make the variable outside the loop to save gas                                                                   |     2     |        -        |
| [G&#x2011;09] | Use assembly to check for address(0)                                                                                 |     3     |        -        |
| [G&#x2011;10] | Before some functions, we should `check some variables` for possible gas save                                        |     9     |        -        |
| [G&#x2011;11] | The result of function calls should be cached rather than re-calling the function                                    |    11     |        -        |
| [G&#x2011;12] | Copying struct to memory can be more expensive than just reading from storage                                        |     1     |      2100       |
| [G&#x2011;13] | Use nested if and, avoid multiple check combinations                                                                 |     4     |        -        |
| [G&#x2011;14] | Multiple accesses of a `mapping/array` should use a local variable cache                                             |     3     |        -        |

Total: 109 instances over 14 issues

## Gas Optimizations

## [G-01] Avoid `contract existence` checks by using solidity version 0.8.10 or later

### Summary

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

### Details

There are **13** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Position.sol

/// @audit checkQualified()
111        IReserve(zchf.reserve()).checkQualified(msg.sender, helpers);


/// @audit balanceOf()
170        return IERC20(collateral).balanceOf(address(this));


/// @audit  transferFrom()
228        IERC20(zchf).transferFrom(msg.sender, address(this), amount);


/// @audit  burnWithReserve()
233        uint256 actuallyBurned = IFrankencoin(zchf).burnWithReserve(burnable, reserveContribution);


/// @audit  transfer()
253            IERC20(token).transfer(target, amount);


/// @audit  transfer()
269        IERC20(collateral).transfer(target, amount);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

```solidity
File: /contracts/MintingHub.sol

/// @audit   transferFrom()
110        IERC20(_collateralAddress).transferFrom(msg.sender, address(pos), _initialCollateral);


/// @audit  initializeClone()
130        IPosition(pos).initializeClone(msg.sender, existing.price(), limit, _initialCollateral, _initialMint);


/// @audit  transferFrom()
142        IERC20(position.collateral()).transferFrom(msg.sender, address(this), _collateralAmount);


/// @audit  minimumCollateral()
170        uint256 min = IPosition(challenge.position).minimumCollateral();


/// @audit transfer()
263            IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);


/// @audit transfer()
284        IERC20(collateral).transfer(target, amount);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

```solidity
File: /contracts/ERC20.sol

/// @audit onTokenTransfer()
165            success = IERC677Receiver(recipient).onTokenTransfer(msg.sender, amount, data);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol

## [G-02] Add `unchecked {}` for subtractions where the operands can't underflow because of previous require() and if() statement

### Summary

In a case where there's previous require and it is certain there will be no underflow, use the unchecker.
This will stop the check for overflow and underflow so it will save gas.

### Details

There are **12** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Position.sol

/// @audit  newCollateral, colbal
138            collateral.transferFrom(msg.sender, address(this), newCollateral - colbal);


/// @audit  colbal, newCollateral
146            withdrawCollateral(msg.sender, colbal - newCollateral);


/// @audit  newMinted, minted
150            mint(msg.sender, newMinted - minted);


/// @audit  minted, amount
242        minted -= amount;


/// @audit
295        challengedAmount += size;


/// @audit  challengedAmount, _collateralAmount
309            challengedAmount -= _collateralAmount;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

```solidity
File: /contracts/MintingHub.sol

/// @audit  effectiveBid, fundsNeeded
268            zchf.transfer(owner, effectiveBid - fundsNeeded);


/// @audit  fundsNeeded, effectiveBid
270            zchf.notifyLoss(fundsNeeded - effectiveBid); // ensure we have enough to pay everything
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

```solidity
File: /contracts/Frankencoin.sol

/// @audit
209         return theoreticalReserve * currentReserve / minterReserve();
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

```solidity
File: /contracts/ERC20.sol

/// @audit  currentAllowance, amount
132            _approve(sender, msg.sender, currentAllowance - amount);


/// @audit  _balances[sender], amount
156        _balances[sender] -= amount;


/// @audit  _balances[recipient], amount
157        _balances[recipient] += amount;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol

## [G-03] Using fixed bytes is cheaper than using string

### Summary

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

### Details

There are **4** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Equity.sol

/// @audit  string
97    function name() override external pure returns (string memory) {


/// @audit  string
101    function symbol() override external pure returns (string memory) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

```solidity
File: /contracts/Frankencoin.sol

/// @audit  string
64   function name() override external pure returns (string memory){


/// @audit  string
68   function symbol() override external pure returns (string memory){
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

## [G-04] Multiple address/id mappings can be combined into a `single mapping` of and address/id to a struct where appropriate

### Summary

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

### Details

There are **8** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/MintingHub.sol

37    mapping (address /** col */ => mapping (address => uint256)) public pendingReturns;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

```solidity
File: /contracts/Equity.sol

83    mapping (address => address) public delegates;

88    mapping (address => uint64) private voteAnchor; // 40 Bit for the block number, 24 Bit sub-block time resolution
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

```solidity
File: /contracts/Frankencoin.sol

45   mapping (address => uint256) public minters;

50   mapping (address => address) public positions;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

```solidity
File: /contracts/ERC20.sol

43    mapping (address => uint256) private _balances;

45    mapping (address => mapping (address => uint256)) private _allowances;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol

```solidity
File: /contracts/ERC20PermitLight.sol

15    mapping(address => uint256) public nonces;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol

### Recommendation Code

```solidity
File: /contracts/MintingHub.sol

37    mapping (address /** col */ => mapping (address => uint256)) public pendingReturns;

struct someName {
    address
    uint256
}

mapping (address /** col */  => someName) public pendingReturns;
```

## [G-05] State variables should be cached in `stack variables` rather than re-reading them from `storage`

### Summary

### Details

There are **40** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Position.sol

/// @audit  minimumCollateral
77        if(_coll < minimumCollateral) revert InsufficientCollateral();

/// @audit  price
81        if (price > _price) revert InsufficientCollateral();

/// @audit  original, zchf, collateral
85        emit PositionOpened(owner, original, address(zchf), address(collateral), _price);

/// @audit  start
110        if (block.timestamp >= start) revert TooLate();

/// @audit  price
133        if (newPrice != price){

/// @audit  minted
141        if (newMinted < minted){

/// @audit  minted
142            zchf.burnFrom(msg.sender, minted - newMinted, reserveContribution);

/// @audit  minted
149        if (newMinted > minted){

/// @audit  price
160        if (newPrice > price) {

/// @audit  start,  start, mintingFeePPM, mintingFeePPM
187            return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));


/// @audit  minted, limit
194..        if (minted + amount > limit) revert LimitExceeded();

/// @audit  cooldown
204        if (horizon > cooldown){

/// @audit  minted
241        if (amount > minted) revert RepaidTooMuch(amount - minted);

/// @audit  collateral
250        if (token == address(collateral)){

/// @audit  minimumCollateral
271        if (balance < minimumCollateral){

/// @audit  minted
283        if (collateralReserve * atPrice < minted * ONE_DEC18) revert InsufficientCollateral();

/// @audit  minted, price,  limit
287...        emit MintingUpdate(collateralBalance(), price, minted, limit);

/// @audit  minimumCollateral
294        if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();

/// @audit  expiration
305        if (block.timestamp >= expiration){

/// @audit  price
307        } else if (_bidAmountZCHF * ONE_DEC18 >= price * _collateralAmount){

/// @audit  expiration
367        if (block.timestamp > expiration) revert Expired();

/// @audit  cooldown
374        if (block.timestamp <= cooldown) revert Hot();

/// @audit  challengedAmount
381        if (challengedAmount > 0) revert Challenged();

/// @audit  hub
388        if (msg.sender != address(hub)) revert NotHub();
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

```solidity
File: /contracts/Equity.sol

/// @audit QUORUM
211        if (_votes * 10000 < QUORUM * totalVotes()) revert NotQualified();

/// @audit  MINIMUM_EQUITY
244        require(equity >= MINIMUM_EQUITY, "insuf equity"); // ensures that the initial deposit is at least 1000 ZCHF

/// @audit  MINIMUM_EQUITY
310        require(zchf.equity() < MINIMUM_EQUITY);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

```solidity
File: /contracts/Frankencoin.sol

///@audit   MIN_APPLICATION_PERIOD
84      if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();

///@audit   MIN_FEE
85      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

```solidity
File: /contracts/ERC20.sol

/// @audit   INFINITY
128        if (currentAllowance < INFINITY){
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol

```solidity
File: /contracts/StablecoinBridge.sol

/// @audit   chf
76        if (msg.sender == address(chf)){

/// @audit   zchf
78        } else if (msg.sender == address(zchf)){
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol

## [G-06] Stack variable used as a cheaper cache for `state variables` is `only used once`

### Summary

If the variable is only accessed once, it's cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend:

### Details

There are **8** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/MintingHub.sol

/// @audit OPENING_FEE
108        zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE);

/// @audit CHALLENGER_REWARD
265        uint256 reward = (volume * CHALLENGER_REWARD) / 1000_000;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

```solidity
File: /contracts/Equity.sol

/// @audit VALUATION_FACTOR
109         return VALUATION_FACTOR * zchf.equity() * ONE_DEC18 / totalSupply();

/// @audit  QUORUM
211        if (_votes * 10000 < QUORUM * totalVotes()) revert NotQualified();


/// @audit  BLOCK_TIME_RESOLUTION_BITS
173        return uint64(block.number << BLOCK_TIME_RESOLUTION_BITS);


/// @audit  MIN_HOLDING_DURATION
136        return anchorTime() - voteAnchor[owner] >= MIN_HOLDING_DURATION;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

```solidity
File: /contracts/Frankencoin.sol


/// @audit   MIN_FEE
25      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

```solidity
File: /contracts/ERC20.sol

/// @audit  INFINITY
128        if (currentAllowance < INFINITY){
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol

### Recommendation Code

```solidity
File: /contracts/MintingHub.sol


- 26      uint32 public constant CHALLENGER_REWARD = 20000; // 2%

- 265        uint256 reward = (volume * CHALLENGER_REWARD) / 1000_000;


+ 265       uint256 reward = (volume * 20000) / 1000_000;
```

## [G-07] Not Using the named `return` variables when a function returns, `WASTES DEPLOYMENT GAS`

### Summary

Do not use return at the end of the function:

### Details

There are **1** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Position.sol

/// @audit result
37    function createClone(address target) internal returns (address result) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

## [G-08] Can make the variable outside the loop to save gas

### Summary

Consider making the stack variables before the loop which gonna save gas

### Details

There are **2** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Equity.sol

193            address current = helpers[i];

313            address current = addressesToWipe[0];
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

## [G-09] Use assembly to check for address(0)

### Summary

### Details

There are **3** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/ERC20.sol

152        require(recipient != address(0));

180        require(recipient != address(0));
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol

```solidity
File: /contracts/ERC20PermitLight.sol

56            require(recoveredAddress != address(0) && recoveredAddress == owner, "INVALID_SIGNER");
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol

## [G-10] Before some functions, we should `check some variables` for possible gas save

### Summary

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:

### Details

There are **9** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Position.sol

///@audit amount
228        IERC20(zchf).transferFrom(msg.sender, address(this), amount);

253            IERC20(token).transfer(target, amount);

269        IERC20(collateral).transfer(target, amount);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

```solidity
File: /contracts/MintingHub.sol

/// @audit OPENING_FEE
108        zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE);


/// @audit _initialCollateral
129        existing.collateral().transferFrom(msg.sender, address(pos), _initialCollateral);

/// @audit _collateralAmount
142        IERC20(position.collateral()).transferFrom(msg.sender, address(this), _collateralAmount);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

```solidity
File: /contracts/StablecoinBridge.sol

/// @audit amount
45        chf.transferFrom(msg.sender, address(this), amount);

68        zchf.burn(zchfHolder, amount);

69        chf.transfer(target, amount);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol

## [G-11] The result of function calls should be cached rather than re-calling the function

### Details

There are **11** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/MintingHub.sol

/// @audit tryAvertChallenge()
208        if (challenge.position.tryAvertChallenge(challenge.size, _bidAmountZCHF)) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

```solidity
File: /contracts/Equity.sol

/// @audit  canVoteFor()
195            require(canVoteFor(sender, current));

/// @audit totalVotes()
211        if (_votes * 10000 < QUORUM * totalVotes()) revert NotQualified();


/// @audit totalSupply()
253        require(totalSupply() < 2**128, "total supply exceeded");

/// @audit canRedeem()
276        require(canRedeem(msg.sender));

/// @audit zchf.equity()
310        require(zchf.equity() < MINIMUM_EQUITY);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

```solidity
File: /contracts/Frankencoin.sol

/// @audit  totalSupply()
84      if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();

/// @audit  totalSupply()
85      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();

/// @audit  isMinter()
106      } else if (isMinter(spender) || isMinter(isPosition(spender))){

/// @audit  isMinter()
126      if (!isMinter(msg.sender)) revert NotMinter();

/// @audit  isMinter()
267      if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

## [G-12] Copying struct to memory can be more expensive than just reading from storage

### Summary

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from
storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array.

If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to
be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

### Details

There are **1** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/MintingHub.sol

/// @audit memory
159        Challenge memory copy = Challenge(
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L159

## [G-13]  Use nested if and, avoid multiple check combinations

### Summary

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

### Details

There are **4** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Position.sol

294        if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

```solidity
File: /contracts/Frankencoin.sol

84      if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();

85      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();

267      if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

### Recommendation Code

```solidity
File: /contracts/Position.sol

- 294        if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();

+          if (size < minimumCollateral){
+             if (size < collateralBalance()) {
+            }
+       }
```

## [G-14] Multiple accesses of a mapping/array should use a local variable cache

### Summary

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata.

### Details

There are **3** instances of this issue.

### Github Permalinks

```solidity
File: /contracts/Frankencoin.sol

/// @audit  minters[_minter]
86      if (minters[_minter] != 0) revert AlreadyRegistered();

/// @audit  minters[_minter]
153      if (block.timestamp > minters[_minter]) revert TooLate();
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

```solidity
File: /contracts/ERC20.sol

/// @audit  _balances[sender]
155        if (_balances[sender] < amount) revert ERC20InsufficientBalance(sender, _balances[sender], amount);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol
