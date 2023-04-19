### Table Of Contents
- [FINDINGS](#findings)
- [IF's/require() statements that check input arguments should be at the top of the function](#ifsrequire-statements-that-check-input-arguments-should-be-at-the-top-of-the-function)
  - [Cheaper to check the function parameter before making an external function call](#cheaper-to-check-the-function-parameter-before-making-an-external-function-call)
  - [Move the require statement at the beginning of the function](#move-the-require-statement-at-the-beginning-of-the-function)
  - [Move the if check condition above the function call](#move-the-if-check-condition-above-the-function-call)
- [The result of a function call should be cached rather than re-calling the function](#the-result-of-a-function-call-should-be-cached-rather-than-re-calling-the-function)
  - [Equity.sol.adjustTotalVotes(): Results of anchorTime() should be cached rather than call it twice](#equitysoladjusttotalvotes-results-of-anchortime-should-be-cached-rather-than-call-it-twice)
  - [Frankencoin.sol.suggestMinter(): Result of totalSupply() should be cached here(sad path)](#frankencoinsolsuggestminter-result-of-totalsupply-should-be-cached-heresad-path)
  - [Frankencoin.sol.calculateAssignedReserve(): Results of minterReserve() should be cached](#frankencoinsolcalculateassignedreserve-results-of-minterreserve-should-be-cached)
- [Multiple accesses of a mapping/array should use a local variable cache](#multiple-accesses-of-a-mappingarray-should-use-a-local-variable-cache)
  - [Frankencoin.sol.suggestMinter(): minters\[\_minter\] should be cached in local storage](#frankencoinsolsuggestminter-minters_minter-should-be-cached-in-local-storage)
- [Using unchecked blocks to save gas](#using-unchecked-blocks-to-save-gas)
- [`2**<n>` should be re-written as `type(uint<n>).max`](#2n-should-be-re-written-as-typeuintnmax)
- [Unnecessary casting as variable is already of the same type](#unnecessary-casting-as-variable-is-already-of-the-same-type)
  - [MintingHub.sol.clonePosition(): pos should not be  cast to address as it's declared as an address](#mintinghubsolcloneposition-pos-should-not-be--cast-to-address-as-its-declared-as-an-address)
- [Note: The following have some caviets, we can reduce the deployment size and deployment cost at the expense of execution cost](#note-the-following-have-some-caviets-we-can-reduce-the-deployment-size-and-deployment-cost-at-the-expense-of-runtime-cost)
  - [shorthand if (We can rewrite the following )](#shorthand-if-we-can-rewrite-the-following-)

## FINDINGS
NB: Some functions have been truncated where necessary to just show affected parts of the code
Through out the report some places might be denoted with audit tags to show the actual place affected.

##  IF's/require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L290-L297
### Cheaper to check the function parameter before making an external function call
```solidity
File: /contracts/Equity.sol
290:    function calculateProceeds(uint256 shares) public view returns (uint256) {
291:        uint256 totalShares = totalSupply();
292:        uint256 capital = zchf.equity();
293:        require(shares + ONE_DEC18 < totalShares, "too many shares"); // make sure there is always at least one share
294:        uint256 newTotalShares = totalShares - shares;
295:        uint256 newCapital = _mulD18(capital, _power3(_divD18(newTotalShares, totalShares)));
296:        return capital - newCapital;
297:    }
```
As we have a require statement verifying a functional parameter, it would be cheaper to run this check first before making an external function call.
```diff
diff --git a/contracts/Equity.sol b/contracts/Equity.sol
index 7057ed6..817e37a 100644
--- a/contracts/Equity.sol
+++ b/contracts/Equity.sol
@@ -289,8 +289,8 @@ contract Equity is ERC20PermitLight, MathUtil, IReserve {
      */
     function calculateProceeds(uint256 shares) public view returns (uint256) {
         uint256 totalShares = totalSupply();
-        uint256 capital = zchf.equity();
         require(shares + ONE_DEC18 < totalShares, "too many shares"); // make sure there is always at least one share
+        uint256 capital = zchf.equity();
         uint256 newTotalShares = totalShares - shares;
         uint256 newCapital = _mulD18(capital, _power3(_divD18(newTotalShares, totalShares)));
         return capital - newCapital;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L88-L113
### Move the require statement at the beginning of the function
```solidity
File: /contracts/MintingHub.sol
88:    function openPosition(
89:        address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
90:        uint256 _mintingMaximum, uint256 _initPeriodSeconds, uint256 _expirationSeconds, uint256 _challengeSeconds,
91:        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address) {
//@audit: truncated some chunk here
107:        zchf.registerPosition(address(pos));
108:        zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE);
109:        require(_initialCollateral >= _minCollateral, "must start with min col");
```
We have a require statement that validates some functional parameters. As we would end up reverting if this parameters don't meet the requirements, it's better to check them at the beggining of the function before performing other operations that would just waste gas in case we end up reverting

```diff
diff --git a/contracts/MintingHub.sol b/contracts/MintingHub.sol
index 663b205..0739259 100644
--- a/contracts/MintingHub.sol
+++ b/contracts/MintingHub.sol
@@ -89,6 +89,8 @@ contract MintingHub {
         address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
         uint256 _mintingMaximum, uint256 _initPeriodSeconds, uint256 _expirationSeconds, uint256 _challengeSeconds,
         uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address) {
+        require(_initialCollateral >= _minCollateral, "must start with min col");
+
         IPosition pos = IPosition(
             POSITION_FACTORY.createNewPosition(
                 msg.sender,
@@ -106,7 +108,6 @@ contract MintingHub {
         );
         zchf.registerPosition(address(pos));
         zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE);
-        require(_initialCollateral >= _minCollateral, "must start with min col");
         IERC20(_collateralAddress).transferFrom(msg.sender, address(pos), _initialCollateral);

         return address(pos);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L76-L86
### Move the if check condition above the function call
```solidity
File: /contracts/Position.sol
76:    function initializeClone(address owner, uint256 _price, uint256 _limit, uint256 _coll, uint256 _mint) external onlyHub {
77:        if(_coll < minimumCollateral) revert InsufficientCollateral();
78:        setOwner(owner);
        
80:        price = _mint * ONE_DEC18 / _coll;
81:        if (price > _price) revert InsufficientCollateral();
```
We have an internal function call that simply sets a new owner. We also have a check of `price > _price` that would revert in case that check fails. As this is not dependent on the internal function call, we can do the check first so that incase of a revert on the `if (price > _price) revert InsufficientCollateral();` we wouldn't waste gas doing the internal function call

```diff
diff --git a/contracts/Position.sol b/contracts/Position.sol
index 3e18534..88ecfe5 100644
--- a/contracts/Position.sol
+++ b/contracts/Position.sol
@@ -75,10 +75,11 @@ contract Position is Ownable, IPosition, MathUtil {
      */
     function initializeClone(address owner, uint256 _price, uint256 _limit, uint256 _coll, uint256 _mint) external onlyHub {
         if(_coll < minimumCollateral) revert InsufficientCollateral();
-        setOwner(owner);
-
         price = _mint * ONE_DEC18 / _coll;
         if (price > _price) revert InsufficientCollateral();
+        setOwner(owner);
```


## The result of a function call should be cached rather than re-calling the function

External calls are expensive. Consider caching the following:

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L144-L148
### Equity.sol.adjustTotalVotes(): Results of anchorTime() should be cached rather than call it twice
```solidity
File: /contracts/Equity.sol
144:    function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {
145:        uint256 lostVotes = from == address(0x0) ? 0 : (anchorTime() - voteAnchor[from]) * amount;
146:        totalVotesAtAnchor = uint192(totalVotes() - roundingLoss - lostVotes);
147:        totalVotesAnchorTime = anchorTime();
148:    }
```

```diff
diff --git a/contracts/Equity.sol b/contracts/Equity.sol
index 7057ed6..6344fef 100644
--- a/contracts/Equity.sol
+++ b/contracts/Equity.sol
@@ -142,9 +142,10 @@ contract Equity is ERC20PermitLight, MathUtil, IReserve {
      * @param amount    amount to be sent
      */
     function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {
-        uint256 lostVotes = from == address(0x0) ? 0 : (anchorTime() - voteAnchor[from]) * amount;
+        uint64 _anchorTime = anchorTime();
+        uint256 lostVotes = from == address(0x0) ? 0 : (_anchorTime - voteAnchor[from]) * amount;
         totalVotesAtAnchor = uint192(totalVotes() - roundingLoss - lostVotes);
-        totalVotesAnchorTime = anchorTime();
+        totalVotesAnchorTime = _anchorTime;
     }
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L83-L90
### Frankencoin.sol.suggestMinter(): Result of totalSupply() should be cached here(sad path)
```solidity
File: /contracts/Frankencoin.sol
83:   function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {
84:      if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
85:      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
86:      if (minters[_minter] != 0) revert AlreadyRegistered();
87:      _transfer(msg.sender, address(reserve), _applicationFee);
88:      minters[_minter] = block.timestamp + _applicationPeriod;
89:      emit MinterApplied(_minter, _applicationPeriod, _applicationFee, _message);
90:   }
```

```diff
diff --git a/contracts/Frankencoin.sol b/contracts/Frankencoin.sol
index e9e87dc..556d882 100644
--- a/contracts/Frankencoin.sol
+++ b/contracts/Frankencoin.sol
@@ -81,8 +81,9 @@ contract Frankencoin is ERC20PermitLight, IFrankencoin {
     * minter.
     */
    function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {
-      if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
-      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
+      uint256 _totalSupply = totalSupply();
+      if (_applicationPeriod < MIN_APPLICATION_PERIOD && _totalSupply > 0) revert PeriodTooShort();
+      if (_applicationFee < MIN_FEE  && _totalSupply > 0) revert FeeTooLow();
       if (minters[_minter] != 0) revert AlreadyRegistered();
       _transfer(msg.sender, address(reserve), _applicationFee);
       minters[_minter] = block.timestamp + _applicationPeriod;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L204-L213
### Frankencoin.sol.calculateAssignedReserve(): Results of minterReserve() should be cached 
```solidity
File: /contracts/Frankencoin.sol
204:   function calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) public view returns (uint256) {
205:      uint256 theoreticalReserve = _reservePPM * mintedAmount / 1000000;
206:      uint256 currentReserve = balanceOf(address(reserve));
207:      if (currentReserve < minterReserve()){ //@audit: Initial call
208:         // not enough reserves, owner has to take a loss
209:         return theoreticalReserve * currentReserve / minterReserve();//@audit: second call
210:      } else {
211:         return theoreticalReserve;
212:      }
213:   }
```

```diff
diff --git a/contracts/Frankencoin.sol b/contracts/Frankencoin.sol
index e9e87dc..1f0a60e 100644
--- a/contracts/Frankencoin.sol
+++ b/contracts/Frankencoin.sol
@@ -204,9 +204,10 @@ contract Frankencoin is ERC20PermitLight, IFrankencoin {
    function calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) public view returns (uint256) {
       uint256 theoreticalReserve = _reservePPM * mintedAmount / 1000000;
       uint256 currentReserve = balanceOf(address(reserve));
-      if (currentReserve < minterReserve()){
+      uint256 _minterReserve = minterReserve();
+      if (currentReserve < _minterReserve){
          // not enough reserves, owner has to take a loss
-         return theoreticalReserve * currentReserve / minterReserve();
+         return theoreticalReserve * currentReserve / _minterReserve;
       } else {
          return theoreticalReserve;
       }
```

## Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time.
**Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it**

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. 
As an example, instead of repeatedly calling ```someMap[someIndex]```, save its reference like this: ```SomeStruct storage someStruct = someMap[someIndex]``` and use it.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L83-L90
### Frankencoin.sol.suggestMinter(): minters[\_minter] should be cached in local storage
```solidity
File: /contracts/Frankencoin.sol
83:   function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {

86:      if (minters[_minter] != 0) revert AlreadyRegistered();//@audit: 1st access
87:      _transfer(msg.sender, address(reserve), _applicationFee);
88:      minters[_minter] = block.timestamp + _applicationPeriod; //@audit: 2nd access
```


## Using unchecked blocks to save gas
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
[see resource](https://github.com/ethereum/solidity/issues/10695)

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L247
```solidity
File: /contracts/Equity.sol
247:        uint256 shares = equity <= amount ? 1000 * ONE_DEC18 : calculateSharesInternal(equity - amount, amount);
```
The operation `equity - amount` cannot underflow as it would only be performed if `equity <= amount`

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L294
```solidity
File: /contracts/Equity.sol
294:        uint256 newTotalShares = totalShares - shares;
```
The above operation cannot underflow due to the check on [Line 293](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L293) which checks that `shares + ONE_DEC18 < totalShares`. This means that  whatever value `shares` would have, the operation would only be performed if it's less than `totalShares`

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L144
```solidity
File: /contracts/Frankencoin.sol
144:        return balance - minReserve;
```
The operation `balance - minReserve` cannot underflow due to the check on [Line 141](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L141) that ensures that `balance` is greater than `minReserve` before performing this operation

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L286
```solidity
File: /contracts/Frankencoin.sol
286:         _mint(msg.sender, _amount - reserveLeft);
```
The operation `_amount - reserveLeft` cannot underflow due to the check on [Line 282](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L282) which ensures that our arithmetic operation would only be performed if `reserveLeft` is less than `_amount`

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L138
```solidity
File: /contracts/Position.sol
138:            collateral.transferFrom(msg.sender, address(this), newCollateral - colbal);
```
The operation `newCollateral - colbal` cannot underflow due to the check on [Line 137](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L137) that ensures that `newCollateral` is greater than `colbal` before performing the arithmetic operation

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L142
```solidity
File: /contracts/Position.sol
142:            zchf.burnFrom(msg.sender, minted - newMinted, reserveContribution);
```
The operation `minted - newMinted` cannot underflow due to the check on [Line 141](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L141) that ensures that `minted` is greater than `newMinted` before performing the arithmetic operation

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L146
```solidity
File: /contracts/Position.sol
146:            withdrawCollateral(msg.sender, colbal - newCollateral);
```
The operation `colbal - newCollateral` cannot underflow due to the check on [Line 145](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L145) that ensures that `newCollateral` is greater than `colbal` before performing the arithmetic operation

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L150
```solidity
File: /contracts/Position.sol
150:            mint(msg.sender, newMinted - minted);
```
The operation `newMinted - minted` cannot underflow due to the check on [Line 149](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L149) that ensures that `newMinted` is greater than `minted` before performing the arithmetic operation

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L240-L243
```solidity
File: /contracts/Position.sol
240:    function notifyRepaidInternal(uint256 amount) internal {
241:        if (amount > minted) revert RepaidTooMuch(amount - minted);
242:        minted -= amount;
243:    }
```

There are two operations here `amount - minted` and ` minted -= amount` , The two operations cannot underflow as they are both protected by the check `if (amount > minted) `. The first one would only be performed during a revert which would be as a result of `amount` being greater than `minted`
The second operation would be performed if we don't hit the revert condition which would mean `minted` was greater than `amount`

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L263
```solidity
File: /contracts/MintingHub.sol
263:            IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);
```
The operation `challenge.bid - effectiveBid` cannot underflow due to the check on [Line 261](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L261) that ensures that `challenge.bid` is greater than `effectiveBid` before performing the arithmetic operation

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L268
```solidity
File: /contracts/MintingHub.sol
268:            zchf.transfer(owner, effectiveBid - fundsNeeded);
```
The operation `effectiveBid - fundsNeeded` cannot underflow due to the check on [Line 267](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L267) that ensures that `effectiveBid` is greater than `fundsNeeded` before performing the arithmetic operation

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L270
```solidity
File: /contracts/MintingHub.sol
270:            zchf.notifyLoss(fundsNeeded - effectiveBid); // ensure we have enough to pay everything
```
The operation `fundsNeeded - effectiveBid` cannot underflow due to the check on [Line 269](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L269) that ensures that `effectiveBid` is less than `fundsNeeded` before performing the arithmetic operation

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L132
```solidity
File: /contracts/ERC20.sol
132:            _approve(sender, msg.sender, currentAllowance - amount);
```
The operation `currentAllowance - amount` cannot underflow due to the check on [Line 131](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L131) that ensures that this arithmetic operation would only be performed if `currentAllowance` is greater than `amount`

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L156
```solidity
File: /contracts/ERC20.sol
156:        _balances[sender] -= amount;
```
The above operation cannot underflow as we have a check on  [Line 155](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L155) that ensures that 
`_balances[sender]` cannot be less than `amount` before we perform the subtractio



## `2**<n>` should be re-written as `type(uint<n>).max`

Earlier versions of solidity can use `uint<n>(-1)` instead. Expressions not including the `- 1` can often be re-written to accomodate the change (e.g. by using a `>` rather than a `>=`, which will also save some gas)
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L241-L255

```solidity
File: /contracts/Equity.sol
241:    function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool) {

253:        require(totalSupply() < 2**128, "total supply exceeded");
254:        return true;
255:    }
```
As our operation didn't include the `-1` we've changed the sign to `<=` rather than just `<`
```diff
diff --git a/contracts/Equity.sol b/contracts/Equity.sol
index 7057ed6..95816f3 100644
--- a/contracts/Equity.sol
+++ b/contracts/Equity.sol
@@ -250,7 +250,7 @@ contract Equity is ERC20PermitLight, MathUtil, IReserve {

         // limit the total supply to a reasonable amount to guard against overflows with price and vote calculations
         // the 128 bits are 68 bits for magnitude and 60 bits for precision, as calculated in an above comment
-        require(totalSupply() < 2**128, "total supply exceeded");
+        require(totalSupply() <= type(uint128).max , "total supply exceeded");
         return true;
     }

```


## Unnecessary casting as variable is already of the same type
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L124-L132
### MintingHub.sol.clonePosition(): pos should not be  cast to address as it's declared as an address
```solidity
File: /contracts/MintingHub.sol
124:    function clonePosition(address position, uint256 _initialCollateral, uint256 _initialMint) public validPos(position) returns (address) {
125:        IPosition existing = IPosition(position);
126:        uint256 limit = existing.reduceLimitForClone(_initialMint);
127:        address pos = POSITION_FACTORY.clonePosition(position);
128:        zchf.registerPosition(pos);
129:        existing.collateral().transferFrom(msg.sender, address(pos), _initialCollateral);
130:        IPosition(pos).initializeClone(msg.sender, existing.price(), limit, _initialCollateral, _initialMint);
131:        return address(pos);
132:    }
```

```diff
diff --git a/contracts/MintingHub.sol b/contracts/MintingHub.sol
index 663b205..c69c0c2 100644
--- a/contracts/MintingHub.sol
+++ b/contracts/MintingHub.sol
@@ -126,9 +126,9 @@ contract MintingHub {
         uint256 limit = existing.reduceLimitForClone(_initialMint);
         address pos = POSITION_FACTORY.clonePosition(position);
         zchf.registerPosition(pos);
-        existing.collateral().transferFrom(msg.sender, address(pos), _initialCollateral);
+        existing.collateral().transferFrom(msg.sender, pos, _initialCollateral);
         IPosition(pos).initializeClone(msg.sender, existing.price(), limit, _initialCollateral, _initialMint);
-        return address(pos);
+        return pos;
     }
```



## Note: The following have some caviets, we can reduce the deployment size and deployment cost at the expense of execution cost

### shorthand if (We can rewrite the following )
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L204-L213
```solidity
File: /contracts/Frankencoin.sol
204:   function calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) public view returns (uint256) {
205:      uint256 theoreticalReserve = _reservePPM * mintedAmount / 1000000;
206:      uint256 currentReserve = balanceOf(address(reserve));
207:      if (currentReserve < minterReserve()){
208:         // not enough reserves, owner has to take a loss
209:         return theoreticalReserve * currentReserve / minterReserve();
210:      } else {
211:         return theoreticalReserve;
212:      }
213:   }
```

```diff
diff --git a/contracts/Frankencoin.sol b/contracts/Frankencoin.sol
index e9e87dc..600b805 100644
--- a/contracts/Frankencoin.sol
+++ b/contracts/Frankencoin.sol
@@ -204,12 +204,7 @@ contract Frankencoin is ERC20PermitLight, IFrankencoin {
    function calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) public view returns (uint256) {
       uint256 theoreticalReserve = _reservePPM * mintedAmount / 1000000;
       uint256 currentReserve = balanceOf(address(reserve));
-      if (currentReserve < minterReserve()){
-         // not enough reserves, owner has to take a loss
-         return theoreticalReserve * currentReserve / minterReserve();
-      } else {
-         return theoreticalReserve;
-      }
+      return currentReserve < minterReserve() ? theoreticalReserve * currentReserve / minterReserve() : theoreticalReserve;
    }
```


https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L138-L146
```solidity
File: /contracts/Frankencoin.sol
138:   function equity() public view returns (uint256) {
139:      uint256 balance = balanceOf(address(reserve));
140:      uint256 minReserve = minterReserve();
141:      if (balance <= minReserve){
142:        return 0;
143:      } else {
144:        return balance - minReserve;
145:      }
146:    }
```

```diff
diff --git a/contracts/Frankencoin.sol b/contracts/Frankencoin.sol
index e9e87dc..f9fef16 100644
--- a/contracts/Frankencoin.sol
+++ b/contracts/Frankencoin.sol
@@ -138,11 +138,7 @@ contract Frankencoin is ERC20PermitLight, IFrankencoin {
    function equity() public view returns (uint256) {
       uint256 balance = balanceOf(address(reserve));
       uint256 minReserve = minterReserve();
-      if (balance <= minReserve){
-        return 0;
-      } else {
-        return balance - minReserve;
-      }
+      return balance <= minReserve ? 0 : balance - minReserve;
     }
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L120-L126
```solidity
File: /contracts/Position.sol
120:    function getUsableMint(uint256 totalMint, bool afterFees) public view returns (uint256){
121:        if (afterFees){
122:            return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
123:        } else {
124:            return totalMint * (1000_000 - reserveContribution) / 1000_000;
125:        }
126:    }
```

```diff
diff --git a/contracts/Position.sol b/contracts/Position.sol
index 3e18534..15183e9 100644
--- a/contracts/Position.sol
+++ b/contracts/Position.sol
@@ -118,11 +118,7 @@ contract Position is Ownable, IPosition, MathUtil {
      * to buy reserve pool shares.
      */
     function getUsableMint(uint256 totalMint, bool afterFees) public view returns (uint256){
-        if (afterFees){
-            return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
-        } else {
-            return totalMint * (1000_000 - reserveContribution) / 1000_000;
-        }
+        return afterFees? totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000 : totalMint * (1000_000 - reserveContribution) / 1000_000;
     }
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L181-L189
```solidity
File: /contracts/Position.sol
181:    function calculateCurrentFee() public view returns (uint32) {

184:        if (time >= exp){
185:            return 0;
186:        } else {
187:            return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));
188:        }
189:    }
```

```diff
diff --git a/contracts/Position.sol b/contracts/Position.sol
index 3e18534..9adc148 100644
--- a/contracts/Position.sol
+++ b/contracts/Position.sol
@@ -181,11 +181,7 @@ contract Position is Ownable, IPosition, MathUtil {
     function calculateCurrentFee() public view returns (uint32) {
         uint256 exp = expiration;
         uint256 time = block.timestamp;
-        if (time >= exp){
-            return 0;
-        } else {
-            return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));
-        }
+        return time >= exp ? 0 : uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));
     }
```
