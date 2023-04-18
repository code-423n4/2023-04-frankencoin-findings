# Summary
A majority of the optimizations were benchmarked via the protocol's tests, i.e. using the following config: `solc version 0.8.13`, `optimizer on`, and `200 runs`. Optimizations that were not benchmarked are explained via EVM gas costs and opcodes.

Below are the overall average gas savings for the following tested functions, with all the optimizations applied:
| Function |    Before   |    After   | Avg Gas Savings |
| ------ | -------- | -------- | ------- |
| Equity.transfer |  85662  |  85527  |  135 | 
| Frankencoin.mint |  117324  |  116933  |  391 | 
| Frankencoin.transfer |  40359  |  40224  |  135 | 
| MintingHub.bid |  99969  |  97084  |  2885 |  
| MintingHub.end |  128303  |  121752  | 6551 | 
| MintingHub.launchChallenge |  211011  | 210751 | 260 | 
| Position.repay |  87902  |  82794  | 5108 | 
| Position.withdrawCollateral |  73058  |  69071  | 3987 | 
| StablecoinBridge.mint |  98219  |  97904  | 315 |  
| TestToken.transfer |  51749  |  51614  | 135 |  

**Total gas saved across all listed functions: 19902**

*Notes*: 

- The [Gas report](#gasreport-output-with-all-optimizations-applied) output, after all optimizations have been applied, can be found at the end of the report.
- The final diffs for each contract, with all the optimizations applied, can be found [here](https://gist.github.com/0xJCN/8c6388a742a495a024eef2ab7271d717).
- Some code snippets may be truncated to save space. Code snippets may also be accompanied by `@audit` tags in comments to aid in explaining the issue.

## Gas Optimizations
| Number |Issue|Instances|
|-|:-|:-:|
| [G-01](#a-mapping-is-more-efficient-than-an-array) | A mapping is more efficient than an array | - | 
| [G-02](#state-variables-can-be-cached-instead-of-re-reading-them-from-storage) | State variables can be cached instead of re-reading them from storage | 34 |
| [G-03](#refactor-code-to-avoid-re-reading-from-storage) | Refactor code to avoid re-reading from storage | 4 | 
| [G-04](#cache-return-value-from-external-call-to-avoid-an-unnecessary-callstaticcall) | Cache return value from external call to avoid an unnecessary CALL/STATICCALL | 1 | 
| [G-05](#refactor-event-to-avoid-emitting-unnecessary-storage-values-that-dont-change) | Refactor event to avoid emitting unnecessary storage values that don't change | - | 
| [G-06](#use-unchecked-for-expressions-that-can-not-underflow-due-to-previous-ifrequire-statement) | Use `unchecked` for expressions that can not underflow due to previous `if`/`require` statement | 2 | 

## A mapping is more efficient than an array
Fetching data from an array is more expensive than fetching data from a mapping. Fetching data from an array will require iterating over the array until you reach your desired data. When using a mapping you only need to know the `key` in order to fetch the data from the exact slot it is stored in. [Source](https://twitter.com/pcaversaccio/status/1464523336730480640)

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L31

*Gas Savings for `MintingHub.bid`, obtained via protocol's tests: Avg 2133 gas* 

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  76367   |  123570  |  99969  |    4     |
| After  |  74234   |  121437  |  97836  |    4     |

```solidity
File: contracts/MintingHub.sol
31:    Challenge[] public challenges; // list of open challenges
```
```diff
diff --git a/contracts/MintingHub.sol b/contracts/MintingHub.sol
index 663b205..1585927 100644
--- a/contracts/MintingHub.sol
+++ b/contracts/MintingHub.sol
@@ -28,7 +28,6 @@ contract MintingHub {
     IPositionFactory private immutable POSITION_FACTORY; // position contract to clone

     IFrankencoin public immutable zchf; // currency
-    Challenge[] public challenges; // list of open challenges

     /**
      * Map to remember pending postponed collateral returns.
@@ -36,6 +35,10 @@ contract MintingHub {
      */
     mapping (address /** col */ => mapping (address => uint256)) public pendingReturns;

+    mapping(uint256 => Challenge) public challenges;
+
+    uint256 public challengesCount;
+
     struct Challenge {
         address challenger; // the address from which the challenge was initiated
         IPosition position; // the position that was challenged
@@ -140,8 +143,9 @@ contract MintingHub {
     function launchChallenge(address _positionAddr, uint256 _collateralAmount) external validPos(_positionAddr) returns (uint256) {
         IPosition position = IPosition(_positionAddr);
         IERC20(position.collateral()).transferFrom(msg.sender, address(this), _collateralAmount);
-        uint256 pos = challenges.length;
-        challenges.push(Challenge(msg.sender, position, _collateralAmount, block.timestamp + position.challengePeriod(), address(0x0), 0));
+        uint256 pos = challengesCount;
+        challenges[pos] = Challenge(msg.sender, position, _collateralAmount, block.timestamp + position.challengePeriod(), address(0x0), 0);
+        challengesCount = pos + 1;
         position.notifyChallengeStarted(_collateralAmount);
         emit ChallengeStarted(msg.sender, address(position), _collateralAmount, pos);
         return pos;
@@ -170,9 +174,10 @@ contract MintingHub {
         uint256 min = IPosition(challenge.position).minimumCollateral();
         require(challenge.size >= min);
         require(copy.size >= min);
-
-        uint256 pos = challenges.length;
-        challenges.push(copy);
+
+        uint256 pos = challengesCount;
+        challenges[pos] = copy;
+        challengesCount = pos + 1;
         emit ChallengeStarted(challenge.challenger, address(challenge.position), challenge.size, _challengeNumber);
         emit ChallengeStarted(copy.challenger, address(copy.position), copy.size, pos);
         return pos;
```

## State variables can be cached instead of re-reading them from storage
Caching of a state variable replaces each `Gwarmaccess (100 gas)` with a much cheaper stack read.

Total Instances: `34`

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L76-L86

### Cache `_mint * ONE_DEC18 / _coll` to save 1 SLOAD
```solidity
File: contracts/Position.sol
76:    function initializeClone(address owner, uint256 _price, uint256 _limit, uint256 _coll, uint256 _mint) external onlyHub {
77:        if(_coll < minimumCollateral) revert InsufficientCollateral();
78:        setOwner(owner);
79:        
80:        price = _mint * ONE_DEC18 / _coll; 
81:        if (price > _price) revert InsufficientCollateral(); // @audit: unnecessary sload, use expression in above line
82:        limit = _limit;
83:        mintInternal(owner, _mint, _coll);
84:
85:        emit PositionOpened(owner, original, address(zchf), address(collateral), _price);
86:    }
```
```diff
diff --git a/contracts/Position.sol b/contracts/Position.sol
index 3e18534..a81a034 100644
--- a/contracts/Position.sol
+++ b/contracts/Position.sol
@@ -77,8 +77,9 @@ contract Position is Ownable, IPosition, MathUtil {
         if(_coll < minimumCollateral) revert InsufficientCollateral();
         setOwner(owner);

-        price = _mint * ONE_DEC18 / _coll;
-        if (price > _price) revert InsufficientCollateral();
+        uint256 _newPrice = _mint * ONE_DEC18 / _coll;
+        price = _newPrice;
+        if (_newPrice > _price) revert InsufficientCollateral();
         limit = _limit;
         mintInternal(owner, _mint, _coll);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L97-L101

### Cache `limit` to save 1 SLOAD
```solidity
File: contracts/Position.sol
97:    function reduceLimitForClone(uint256 _minimum) external noChallenge noCooldown alive onlyHub returns (uint256) {
98:        uint256 reduction = (limit - minted - _minimum)/2; // this will fail with an underflow if minimum is too high // @audit: 1st sload
99:        limit -= reduction + _minimum; // @audit: 2nd sload
100:       return reduction + _minimum;
101:   }
```
```diff
diff --git a/contracts/Position.sol b/contracts/Position.sol
index 3e18534..269d1f4 100644
--- a/contracts/Position.sol
+++ b/contracts/Position.sol
@@ -95,8 +95,9 @@ contract Position is Ownable, IPosition, MathUtil {
      * @return limit for the clone
      */
     function reduceLimitForClone(uint256 _minimum) external noChallenge noCooldown alive onlyHub returns (uint256) {
-        uint256 reduction = (limit - minted - _minimum)/2; // this will fail with an underflow if minimum is too high
-        limit -= reduction + _minimum;
+        uint256 _limit = limit;
+        uint256 reduction = (_limit - minted - _minimum)/2; // this will fail with an underflow if minimum is too high
+        limit = _limit - (reduction + _minimum);
         return reduction + _minimum;
     }
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L132-L152

### Cache `minted` to save 3 SLOADs
```solidity
File: contracts/Position.sol
132:    function adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) public onlyOwner {
133:        if (newPrice != price){
134:            adjustPrice(newPrice);
135:        }
136:        uint256 colbal = collateralBalance();
137:        if (newCollateral > colbal){
138:            collateral.transferFrom(msg.sender, address(this), newCollateral - colbal);
139:        }
140:        // Must be called after collateral deposit, but before withdrawal
141:        if (newMinted < minted){ // @audit: 1st sload
142:            zchf.burnFrom(msg.sender, minted - newMinted, reserveContribution); // @audit: 2nd sload
143:            minted = newMinted;
144:        }
145:        if (newCollateral < colbal){
146:            withdrawCollateral(msg.sender, colbal - newCollateral);
147:        }
148:        // Must be called after collateral withdrawal
149:        if (newMinted > minted){ // @audit: 3rd sload
150:            mint(msg.sender, newMinted - minted); // @audit: 4th sload
151:        }
152:    }
```
```diff
diff --git a/contracts/Position.sol b/contracts/Position.sol
index 3e18534..76de4c2 100644
--- a/contracts/Position.sol
+++ b/contracts/Position.sol
@@ -138,16 +138,17 @@ contract Position is Ownable, IPosition, MathUtil {
             collateral.transferFrom(msg.sender, address(this), newCollateral - colbal);
         }
         // Must be called after collateral deposit, but before withdrawal
-        if (newMinted < minted){
-            zchf.burnFrom(msg.sender, minted - newMinted, reserveContribution);
+        uint256 _minted = minted;
+        if (newMinted < _minted){
+            zchf.burnFrom(msg.sender, _minted - newMinted, reserveContribution);
             minted = newMinted;
         }
         if (newCollateral < colbal){
             withdrawCollateral(msg.sender, colbal - newCollateral);
         }
         // Must be called after collateral withdrawal
-        if (newMinted > minted){
-            mint(msg.sender, newMinted - minted);
+        if (newMinted > _minted){
+            mint(msg.sender, newMinted - _minted);
         }
     }
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L193-L200

### Cache `minted` to save 1 SLOAD
```solidity
File: contracts/Position.sol
193:    function mintInternal(address target, uint256 amount, uint256 collateral_) internal {
194:        if (minted + amount > limit) revert LimitExceeded(); // @audit: 1st sload
195:        zchf.mint(target, amount, reserveContribution, calculateCurrentFee());
196:        minted += amount; // @audit: 2nd sload
197:
198:        checkCollateral(collateral_, price);
199:        emitUpdate();
200:    }
```
```diff
diff --git a/contracts/Position.sol b/contracts/Position.sol
index 3e18534..1fdb814 100644
--- a/contracts/Position.sol
+++ b/contracts/Position.sol
@@ -191,9 +191,10 @@ contract Position is Ownable, IPosition, MathUtil {
     error LimitExceeded();

     function mintInternal(address target, uint256 amount, uint256 collateral_) internal {
-        if (minted + amount > limit) revert LimitExceeded();
+        uint256 _minted = minted;
+        if (_minted + amount > limit) revert LimitExceeded();
         zchf.mint(target, amount, reserveContribution, calculateCurrentFee());
-        minted += amount;
+        minted = _minted + amount;

         checkCollateral(collateral_, price);
         emitUpdate();
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L240-L243

### Cache `minted` to save 1 SLOAD
```solidity
File: contracts/Position.sol
240:    function notifyRepaidInternal(uint256 amount) internal {
241:        if (amount > minted) revert RepaidTooMuch(amount - minted); // @audit: 1st sload
242:        minted -= amount; // @audit: 2nd sload
243:    }
```
```diff
diff --git a/contracts/Position.sol b/contracts/Position.sol
index 3e18534..f63cc33 100644
--- a/contracts/Position.sol
+++ b/contracts/Position.sol
@@ -238,8 +238,9 @@ contract Position is Ownable, IPosition, MathUtil {
     error RepaidTooMuch(uint256 excess);

     function notifyRepaidInternal(uint256 amount) internal {
-        if (amount > minted) revert RepaidTooMuch(amount - minted);
-        minted -= amount;
+        uint256 _minted = minted;
+        if (amount > _minted) revert RepaidTooMuch(amount - _minted);
+        minted = _minted - amount;
     }
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L347-L354

### Cache `minted` to save 1 SLOAD
```solidity
File: contracts/Position.sol
347:        uint256 volumeZCHF = _mulD18(price, _size); // How much could have minted with the challenged amount of the collateral
348:        // The owner does not have to repay (and burn) more than the owner actually minted.  
349:        uint256 repayment = minted < volumeZCHF ? minted : volumeZCHF; // how much must be burned to make things even // @audit: 1st & 2nd sload
350:
351:        notifyRepaidInternal(repayment); // we assume the caller takes care of the actual repayment
352:        internalWithdrawCollateral(_bidder, _size); // transfer collateral to the bidder and emit update
353:        return (owner, _bid, volumeZCHF, repayment, reserveContribution);
354:    }
```
```diff
diff --git a/contracts/Position.sol b/contracts/Position.sol
index 3e18534..4d6196a 100644
--- a/contracts/Position.sol
+++ b/contracts/Position.sol
@@ -346,7 +346,8 @@ contract Position is Ownable, IPosition, MathUtil {
         // So the owner cannot maliciously decrease the price to make volume fall below the proportionate repayment.
         uint256 volumeZCHF = _mulD18(price, _size); // How much could have minted with the challenged amount of the collateral
         // The owner does not have to repay (and burn) more than the owner actually minted.
-        uint256 repayment = minted < volumeZCHF ? minted : volumeZCHF; // how much must be burned to make things even
+        uint256 _minted = minted;
+        uint256 repayment = _minted < volumeZCHF ? _minted : volumeZCHF; // how much must be burned to make things even

         notifyRepaidInternal(repayment); // we assume the caller takes care of the actual repayment
         internalWithdrawCollateral(_bidder, _size); // transfer collateral to the bidder and emit update
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L156-L179

### Cache `challenge.challenger`, `challenge.position`, `challenge.bid`, and `challenge.size` to save 8 SLOADs
```solidity
File: contracts/MintingHub.sol
156:    function splitChallenge(uint256 _challengeNumber, uint256 splitOffAmount) external returns (uint256) {
157:        Challenge storage challenge = challenges[_challengeNumber];
158:        require(challenge.challenger != address(0x0)); // @audit: 1st sload
159:        Challenge memory copy = Challenge(
160:            challenge.challenger, // @audit: 2nd sload
161:            challenge.position, // @audit: 1st sload
162:            splitOffAmount,
163:            challenge.end,
164:            challenge.bidder,
165:            (challenge.bid * splitOffAmount) / challenge.size // @audit: 1st sloads
166:        );
167:        challenge.bid -= copy.bid; // @audit: 2nd sload
168:        challenge.size -= copy.size; // @audit: 2nd sload
169:
170:        uint256 min = IPosition(challenge.position).minimumCollateral(); // @audit: 2nd sload
171:        require(challenge.size >= min); // @audit: 3rd sload
172:        require(copy.size >= min);
173:
174:        uint256 pos = challenges.length;
175:        challenges.push(copy);
176:        emit ChallengeStarted(challenge.challenger, address(challenge.position), challenge.size, _challengeNumber); // @audit: 3rd sload & 4th sload
177:        emit ChallengeStarted(copy.challenger, address(copy.position), copy.size, pos);
178:        return pos;
179:    }
```
```diff
diff --git a/contracts/MintingHub.sol b/contracts/MintingHub.sol
index 663b205..d0c10f7 100644
--- a/contracts/MintingHub.sol
+++ b/contracts/MintingHub.sol
@@ -155,25 +155,29 @@ contract MintingHub {
      */
     function splitChallenge(uint256 _challengeNumber, uint256 splitOffAmount) external returns (uint256) {
         Challenge storage challenge = challenges[_challengeNumber];
-        require(challenge.challenger != address(0x0));
+        address _challenger = challenge.challenger;
+        require(_challenger != address(0x0));
+        IPosition _position = challenge.position;
+        uint256 _bid = challenge.bid;
+        uint256 _size = challenge.size;
         Challenge memory copy = Challenge(
-            challenge.challenger,
-            challenge.position,
+            _challenger,
+            _position,
             splitOffAmount,
             challenge.end,
             challenge.bidder,
-            (challenge.bid * splitOffAmount) / challenge.size
+            (_bid * splitOffAmount) / _size
         );
-        challenge.bid -= copy.bid;
-        challenge.size -= copy.size;
+        challenge.bid = _bid - copy.bid;
+        challenge.size = _size - copy.size;

-        uint256 min = IPosition(challenge.position).minimumCollateral();
-        require(challenge.size >= min);
+        uint256 min = IPosition(_position).minimumCollateral();
+        require(_size >= min);
         require(copy.size >= min);

         uint256 pos = challenges.length;
         challenges.push(copy);
-        emit ChallengeStarted(challenge.challenger, address(challenge.position), challenge.size, _challengeNumber);
+        emit ChallengeStarted(_challenger, address(_position), _size, _challengeNumber);
         emit ChallengeStarted(copy.challenger, address(copy.position), copy.size, pos);
         return pos;
     }
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L199-L229

### Cache `challenge.end`, `challenge.size`, `challenge.bid`, and `challenge.position` to save 6 SLOADs
```solidity
File: contracts/MintingHub.sol
199:    function bid(uint256 _challengeNumber, uint256 _bidAmountZCHF, uint256 expectedSize) external {
200:        Challenge storage challenge = challenges[_challengeNumber];
201:        if (block.timestamp >= challenge.end) revert TooLate(); // @audit: 1st sload
202:        if (expectedSize != challenge.size) revert UnexpectedSize(); // @audit: 1st sload
203:        if (challenge.bid > 0) { // @audit: 1st sload
204:            zchf.transfer(challenge.bidder, challenge.bid); // return old bid // @audit: 2nd sload
205:        }
206:        emit NewBid(_challengeNumber, _bidAmountZCHF, msg.sender);
207:        // ask position if the bid was high enough to avert the challenge
208:        if (challenge.position.tryAvertChallenge(challenge.size, _bidAmountZCHF)) { // @audit: 1st sload & 2nd sload
209:            // bid was high enough, let bidder buy collateral from challenger
210:            zchf.transferFrom(msg.sender, challenge.challenger, _bidAmountZCHF);
211:            challenge.position.collateral().transfer(msg.sender, challenge.size); // @audit: 2nd sload & 3rd sload
212:            emit ChallengeAverted(address(challenge.position), _challengeNumber); // @audit: 3rd sload
213:            delete challenges[_challengeNumber];
214:        } else {
215:            // challenge is not averted, update bid
216:            if (_bidAmountZCHF < minBid(challenge)) revert BidTooLow(_bidAmountZCHF, minBid(challenge));
217:            uint256 earliestEnd = block.timestamp + 30 minutes;
218:            if (earliestEnd >= challenge.end) { // @audit: 2nd sload
219:                // bump remaining time like ebay does when last minute bids come in
220:                // An attacker trying to postpone the challenge forever must increase the bid by 0.5%
221:                // every 30 minutes, or double it every three days, making the attack hard to sustain
222:                // for a prolonged period of time.
223:                challenge.end = earliestEnd;
224:            }
225:            zchf.transferFrom(msg.sender, address(this), _bidAmountZCHF);
226:            challenge.bid = _bidAmountZCHF; // @audit: 3rd sload
227:            challenge.bidder = msg.sender;
228:        }
229:    }
```
```diff
diff --git a/contracts/MintingHub.sol b/contracts/MintingHub.sol
index 663b205..d4aa22f 100644
--- a/contracts/MintingHub.sol
+++ b/contracts/MintingHub.sol
@@ -198,24 +198,28 @@ contract MintingHub {
      */
     function bid(uint256 _challengeNumber, uint256 _bidAmountZCHF, uint256 expectedSize) external {
         Challenge storage challenge = challenges[_challengeNumber];
-        if (block.timestamp >= challenge.end) revert TooLate();
-        if (expectedSize != challenge.size) revert UnexpectedSize();
-        if (challenge.bid > 0) {
-            zchf.transfer(challenge.bidder, challenge.bid); // return old bid
+        uint256 _end = challenge.end;
+        if (block.timestamp >= _end) revert TooLate();
+        uint256 _size = challenge.size;
+        if (expectedSize != _size) revert UnexpectedSize();
+        uint256 _bid = challenge.bid;
+        if (_bid > 0) {
+            zchf.transfer(challenge.bidder, _bid); // return old bid
         }
         emit NewBid(_challengeNumber, _bidAmountZCHF, msg.sender);
         // ask position if the bid was high enough to avert the challenge
-        if (challenge.position.tryAvertChallenge(challenge.size, _bidAmountZCHF)) {
+        IPosition _position = challenge.position;
+        if (_position.tryAvertChallenge(_size, _bidAmountZCHF)) {
             // bid was high enough, let bidder buy collateral from challenger
             zchf.transferFrom(msg.sender, challenge.challenger, _bidAmountZCHF);
-            challenge.position.collateral().transfer(msg.sender, challenge.size);
-            emit ChallengeAverted(address(challenge.position), _challengeNumber);
+            _position.collateral().transfer(msg.sender, _size);
+            emit ChallengeAverted(address(_position), _challengeNumber);
             delete challenges[_challengeNumber];
         } else {
             // challenge is not averted, update bid
             if (_bidAmountZCHF < minBid(challenge)) revert BidTooLow(_bidAmountZCHF, minBid(challenge));
             uint256 earliestEnd = block.timestamp + 30 minutes;
-            if (earliestEnd >= challenge.end) {
+            if (earliestEnd >= _end) {
                 // bump remaining time like ebay does when last minute bids come in
                 // An attacker trying to postpone the challenge forever must increase the bid by 0.5%
                 // every 30 minutes, or double it every three days, making the attack hard to sustain
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L252-L276

### Cache `challenge.challenger`, `challenge.bidder`, `challenge.bid`, and `challenge.position` to save 7 SLOADs
```solidity
File: contracts/MintingHub.sol
252:    function end(uint256 _challengeNumber, bool postponeCollateralReturn) public {
253:        Challenge storage challenge = challenges[_challengeNumber];
254:        require(challenge.challenger != address(0x0)); // @audit: 1st sload
255:        require(block.timestamp >= challenge.end, "period has not ended");
256:        // challenge must have been successful, because otherwise it would have immediately ended on placing the winning bid
257:        returnCollateral(challenge, postponeCollateralReturn);
258:        // notify the position that will send the collateral to the bidder. If there is no bid, send the collateral to msg.sender
259:        address recipient = challenge.bidder == address(0x0) ? msg.sender : challenge.bidder; // @audit: 1st and 2nd sload
260:        (address owner, uint256 effectiveBid, uint256 volume, uint256 repayment, uint32 reservePPM) = challenge.position.notifyChallengeSucceeded(recipient, challenge.bid, challenge.size); // @audit: 1st sloads
261:        if (effectiveBid < challenge.bid) { // @audit: 2nd sload
262:            // overbid, return excess amount
263:            IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid); // @audit: 3rd sload & 2nd sload
264:        }
265:        uint256 reward = (volume * CHALLENGER_REWARD) / 1000_000;
266:        uint256 fundsNeeded = reward + repayment;
267:        if (effectiveBid > fundsNeeded){
268:            zchf.transfer(owner, effectiveBid - fundsNeeded);
269:        } else if (effectiveBid < fundsNeeded){
270:            zchf.notifyLoss(fundsNeeded - effectiveBid); // ensure we have enough to pay everything
271:        }
272:        zchf.transfer(challenge.challenger, reward); // pay out the challenger reward // @audit: 2nd sload
273:        zchf.burn(repayment, reservePPM); // Repay the challenged part
274:        emit ChallengeSucceeded(address(challenge.position), challenge.bid, _challengeNumber); // @audit: 2nd sload & 4th sload
275:        delete challenges[_challengeNumber];
276:    }
```
```diff
diff --git a/contracts/MintingHub.sol b/contracts/MintingHub.sol
index 663b205..d52d678 100644
--- a/contracts/MintingHub.sol
+++ b/contracts/MintingHub.sol
@@ -251,16 +251,27 @@ contract MintingHub {
      */
     function end(uint256 _challengeNumber, bool postponeCollateralReturn) public {
         Challenge storage challenge = challenges[_challengeNumber];
-        require(challenge.challenger != address(0x0));
+        address _challenger = challenge.challenger;
+        require(_challenger != address(0x0));
         require(block.timestamp >= challenge.end, "period has not ended");
         // challenge must have been successful, because otherwise it would have immediately ended on placing the winning bid
         returnCollateral(challenge, postponeCollateralReturn);
         // notify the position that will send the collateral to the bidder. If there is no bid, send the collateral to msg.sender
-        address recipient = challenge.bidder == address(0x0) ? msg.sender : challenge.bidder;
-        (address owner, uint256 effectiveBid, uint256 volume, uint256 repayment, uint32 reservePPM) = challenge.position.notifyChallengeSucceeded(recipient, challenge.bid, challenge.size);
-        if (effectiveBid < challenge.bid) {
-            // overbid, return excess amount
-            IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);
+        IPosition _position = challenge.position;
+        uint256 _bid = challenge.bid;
+        address owner;
+        uint256 effectiveBid;
+        uint256 volume;
+        uint256 repayment;
+        uint32 reservePPM;
+        { // @audit: needed due to `stack too deep error`
+            address _bidder = challenge.bidder;
+            address recipient = _bidder == address(0x0) ? msg.sender : _bidder;
+            (owner, effectiveBid, volume, repayment, reservePPM) = _position.notifyChallengeSucceeded(recipient, _bid, challenge.size);
+            if (effectiveBid < _bid) {
+                // overbid, return excess amount
+                IERC20(zchf).transfer(_bidder, _bid - effectiveBid);
+            }
         }
         uint256 reward = (volume * CHALLENGER_REWARD) / 1000_000;
         uint256 fundsNeeded = reward + repayment;
@@ -269,9 +280,9 @@ contract MintingHub {
         } else if (effectiveBid < fundsNeeded){
             zchf.notifyLoss(fundsNeeded - effectiveBid); // ensure we have enough to pay everything
         }
-        zchf.transfer(challenge.challenger, reward); // pay out the challenger reward
+        zchf.transfer(_challenger, reward); // pay out the challenger reward
         zchf.burn(repayment, reservePPM); // Repay the challenged part
-        emit ChallengeSucceeded(address(challenge.position), challenge.bid, _challengeNumber);
+        emit ChallengeSucceeded(address(_position), _bid, _challengeNumber);
         delete challenges[_challengeNumber];
     }
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L287-L296

### Cache `challenge.challenger` and `challenge.size` to save 3 SLOADs
```solidity
File: contracts/MintingHub.sol
287:    function returnCollateral(Challenge storage challenge, bool postpone) internal {
288:        if (postpone){
289:            // Postponing helps in case the challenger was blacklisted on the collateral token or otherwise cannot receive it at the moment.
290:            address collateral = address(challenge.position.collateral());
291:            pendingReturns[collateral][challenge.challenger] += challenge.size; // @audit: 1st and 2nd sload & 1st sload
292:            emit PostPonedReturn(collateral, challenge.challenger, challenge.size); // @audit: 3rd sload & 2nd sload
293:        } else {
294:            challenge.position.collateral().transfer(challenge.challenger, challenge.size); // return the challenger's collateral
295:        }
296:    }
```
```diff
diff --git a/contracts/MintingHub.sol b/contracts/MintingHub.sol
index 663b205..2e5b107 100644
--- a/contracts/MintingHub.sol
+++ b/contracts/MintingHub.sol
@@ -285,11 +285,14 @@ contract MintingHub {
     }

     function returnCollateral(Challenge storage challenge, bool postpone) internal {
+
         if (postpone){
+            address _challenger = challenge.challenger;
+            uint256 _size = challenge.size;
             // Postponing helps in case the challenger was blacklisted on the collateral token or otherwise cannot receive it at the moment.
             address collateral = address(challenge.position.collateral());
-            pendingReturns[collateral][challenge.challenger] += challenge.size;
-            emit PostPonedReturn(collateral, challenge.challenger, challenge.size);
+            pendingReturns[collateral][_challenger] += _size;
+            emit PostPonedReturn(collateral, _challenger, _size);
         } else {
             challenge.position.collateral().transfer(challenge.challenger, challenge.size); // return the challenger's collateral
         }
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L293-L295

### Cache `minters[_minter]` to save 1 SLOAD

*Function is in a modifier that is used within state mutating functions*
```solidity
File: contracts/Frankencoin.sol
293:   function isMinter(address _minter) override public view returns (bool){
294:      return minters[_minter] != 0 && block.timestamp >= minters[_minter]; // @audit: 1st and 2nd sloads
295:   }
```
```diff
diff --git a/contracts/Frankencoin.sol b/contracts/Frankencoin.sol
index e9e87dc..93b224e 100644
--- a/contracts/Frankencoin.sol
+++ b/contracts/Frankencoin.sol
@@ -291,7 +291,8 @@ contract Frankencoin is ERC20PermitLight, IFrankencoin {
     * Returns true if the address is an approved minter.
     */
    function isMinter(address _minter) override public view returns (bool){
-      return minters[_minter] != 0 && block.timestamp >= minters[_minter];
+       uint256 _minter = minters[_minter];
+      return _minter != 0 && block.timestamp >= _minter;
    }
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L151-L159

### Cache `_balances[sender]` to save 1 SLOAD
```solidity
File: contracts/ERC20.sol
151:    function _transfer(address sender, address recipient, uint256 amount) internal virtual {
152:        require(recipient != address(0));
153:        
154:        _beforeTokenTransfer(sender, recipient, amount);
155:        if (_balances[sender] < amount) revert ERC20InsufficientBalance(sender, _balances[sender], amount); // @audit: 1st sload
156:        _balances[sender] -= amount; // @audit: 2nd sload
157:        _balances[recipient] += amount;
158:        emit Transfer(sender, recipient, amount);
159:    }
```
```diff
diff --git a/contracts/ERC20.sol b/contracts/ERC20.sol
index 585f28b..e588b5b 100644
--- a/contracts/ERC20.sol
+++ b/contracts/ERC20.sol
@@ -152,8 +152,9 @@ abstract contract ERC20 is IERC20 {
         require(recipient != address(0));

         _beforeTokenTransfer(sender, recipient, amount);
-        if (_balances[sender] < amount) revert ERC20InsufficientBalance(sender, _balances[sender], amount);
-        _balances[sender] -= amount;
+        uint256 _senderBalance = _balances[sender];
+        if (_senderBalance < amount) revert ERC20InsufficientBalance(sender, _senderBalance, amount);
+        _balances[sender] = _senderBalance - amount;
         _balances[recipient] += amount;
         emit Transfer(sender, recipient, amount);
     }
```

## Refactor code to avoid re-reading from storage
The internal functions below read storage slots that are previously read in the functions that invoke them. We can refactor the internal functions so we could pass cached storage variables as stack variables and avoid the extra storage reads that would otherwise take place in the internal functions.

Total Instances: `4`

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L227-L243

*Gas Savings for `Position.repay`, obtained via protocol's tests: Avg 207 gas* 

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  86980   |  88824   |  87902  |    2     |
| After  |  86796   |  88594   |  87695  |    2     |

```solidity
File: contracts/Position.sol
240:    function notifyRepaidInternal(uint256 amount) internal {
241:        if (amount > minted) revert RepaidTooMuch(amount - minted); 
242:        minted -= amount;
243:    }
```
```diff
diff --git a/contracts/Position.sol b/contracts/Position.sol
index 3e18534..030ce47 100644
--- a/contracts/Position.sol
+++ b/contracts/Position.sol
@@ -231,15 +231,16 @@ contract Position is Ownable, IPosition, MathUtil {

     function repayInternal(uint256 burnable) internal {
         uint256 actuallyBurned = IFrankencoin(zchf).burnWithReserve(burnable, reserveContribution);
-        notifyRepaidInternal(actuallyBurned);
-        emitUpdate();
+        uint256 _minted = minted;
+        notifyRepaidInternal(actuallyBurned, _minted);
+        emit MintingUpdate(collateralBalance(), price, _minted, limit);
     }

     error RepaidTooMuch(uint256 excess);

-    function notifyRepaidInternal(uint256 amount) internal {
-        if (amount > minted) revert RepaidTooMuch(amount - minted);
-        minted -= amount;
+    function notifyRepaidInternal(uint256 amount, uint256 _minted) internal {
+        if (amount > _minted) revert RepaidTooMuch(amount - _minted);
+        minted = _minted - amount;
     }

     /**
@@ -346,9 +347,10 @@ contract Position is Ownable, IPosition, MathUtil {
         // So the owner cannot maliciously decrease the price to make volume fall below the proportionate repayment.
         uint256 volumeZCHF = _mulD18(price, _size); // How much could have minted with the challenged amount of the collateral
         // The owner does not have to repay (and burn) more than the owner actually minted.
-        uint256 repayment = minted < volumeZCHF ? minted : volumeZCHF; // how much must be burned to make things even
+        uint256 _minted = minted;
+        uint256 repayment = _minted < volumeZCHF ? _minted : volumeZCHF; // how much must be burned to make things even

-        notifyRepaidInternal(repayment); // we assume the caller takes care of the actual repayment
+        notifyRepaidInternal(repayment, _minted); // we assume the caller takes care of the actual repayment
         internalWithdrawCollateral(_bidder, _size); // transfer collateral to the bidder and emit update
         return (owner, _bid, volumeZCHF, repayment, reserveContribution);
     }
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L268-L284

*Gas Savings for `Position.withdrawCollateral`, obtained via protocol's tests: Avg 211 gas* 

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |    -     |    -     |  73058  |    1     |
| After  |    -     |    -     |  72847  |    1     |

```solidity
File: contracts/Position.sol
268:    function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {
269:        IERC20(collateral).transfer(target, amount);
270:        uint256 balance = collateralBalance();
271:        if (balance < minimumCollateral){
272:            cooldown = expiration;
273:        }
274:        emitUpdate();
275:        return balance;
276:    }
277:
278:    /**
279:     * This invariant must always hold and must always be checked when any of the three
280:     * variables change in an adverse way.
281:     */
282:    function checkCollateral(uint256 collateralReserve, uint256 atPrice) internal view {
283:        if (collateralReserve * atPrice < minted * ONE_DEC18) revert InsufficientCollateral();
284:    }
```
```diff
diff --git a/contracts/Position.sol b/contracts/Position.sol
index 3e18534..cf14766 100644
--- a/contracts/Position.sol
+++ b/contracts/Position.sol
@@ -261,17 +261,19 @@ contract Position is Ownable, IPosition, MathUtil {
      * Withdrawing collateral below the minimum collateral amount formally closes the position.
      */
     function withdrawCollateral(address target, uint256 amount) public onlyOwner noChallenge noCooldown {
-        uint256 balance = internalWithdrawCollateral(target, amount);
-        checkCollateral(balance, price);
+        uint256 _price = price;
+        uint256 _minted = minted;
+        uint256 balance = internalWithdrawCollateral(target, amount, _price, _minted);
+        checkCollateral(balance, _price, _minted);
     }

-    function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {
+    function internalWithdrawCollateral(address target, uint256 amount, uint256 _price, uint256 _minted) internal returns (uint256) {
         IERC20(collateral).transfer(target, amount);
         uint256 balance = collateralBalance();
         if (balance < minimumCollateral){
             cooldown = expiration;
         }
-        emitUpdate();
+        emit MintingUpdate(collateralBalance(), _price, _minted, limit);
         return balance;
     }

@@ -279,8 +281,8 @@ contract Position is Ownable, IPosition, MathUtil {
      * This invariant must always hold and must always be checked when any of the three
      * variables change in an adverse way.
      */
-    function checkCollateral(uint256 collateralReserve, uint256 atPrice) internal view {
-        if (collateralReserve * atPrice < minted * ONE_DEC18) revert InsufficientCollateral();
+    function checkCollateral(uint256 collateralReserve, uint256 atPrice, uint256 _minted) internal view {
+        if (collateralReserve * atPrice < _minted * ONE_DEC18) revert InsufficientCollateral();
     }

     function emitUpdate() internal {
@@ -344,12 +346,14 @@ contract Position is Ownable, IPosition, MathUtil {
         // such that
         //    volumeZCHF = price * size / E18 >= minted * size / colbal
         // So the owner cannot maliciously decrease the price to make volume fall below the proportionate repayment.
-        uint256 volumeZCHF = _mulD18(price, _size); // How much could have minted with the challenged amount of the collateral
+        uint256 _price = price;
+        uint256 volumeZCHF = _mulD18(_price, _size); // How much could have minted with the challenged amount of the collateral
         // The owner does not have to repay (and burn) more than the owner actually minted.
-        uint256 repayment = minted < volumeZCHF ? minted : volumeZCHF; // how much must be burned to make things even
+        uint256 _minted = minted;
+        uint256 repayment = _minted < volumeZCHF ? _minted : volumeZCHF; // how much must be burned to make things even

         notifyRepaidInternal(repayment); // we assume the caller takes care of the actual repayment
-        internalWithdrawCollateral(_bidder, _size); // transfer collateral to the bidder and emit update
+        internalWithdrawCollateral(_bidder, _size, _price, _minted); // transfer collateral to the bidder and emit update
         return (owner, _bid, volumeZCHF, repayment, reserveContribution);
     }
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L188-L190

*Gas Savings for `MintingHub.bid`, obtained via protocol's tests: Avg 159 gas* 

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  76367   |  123570  |  99969  |    4     |
| After  |  76157   |  123463  |  99810  |    4     |

```solidity
File: contracts/MintingHub.sol
188:    function minBid(Challenge storage challenge) internal view returns (uint256) {
189:        return (challenge.bid * 1005) / 1000;
190:    }
```
```diff
diff --git a/contracts/MintingHub.sol b/contracts/MintingHub.sol
index 663b205..84aa645 100644
--- a/contracts/MintingHub.sol
+++ b/contracts/MintingHub.sol
@@ -179,14 +179,14 @@ contract MintingHub {
     }

     function minBid(uint256 challenge) public view returns (uint256) {
-        return minBid(challenges[challenge]);
+        return _minBid(challenges[challenge].bid);
     }

     /**
      * The minimum bid size for the next bid. It must be 0.5% higher than the previous bid.
      */
-    function minBid(Challenge storage challenge) internal view returns (uint256) {
-        return (challenge.bid * 1005) / 1000;
+    function _minBid(uint256 _bid) internal view returns (uint256) {
+        return (_bid * 1005) / 1000;
     }

     /**
@@ -200,8 +200,9 @@ contract MintingHub {
         Challenge storage challenge = challenges[_challengeNumber];
         if (block.timestamp >= challenge.end) revert TooLate();
         if (expectedSize != challenge.size) revert UnexpectedSize();
-        if (challenge.bid > 0) {
-            zchf.transfer(challenge.bidder, challenge.bid); // return old bid
+        uint256 _bid = challenge.bid;
+        if (_bid > 0) {
+            zchf.transfer(challenge.bidder, _bid); // return old bid
         }
         emit NewBid(_challengeNumber, _bidAmountZCHF, msg.sender);
         // ask position if the bid was high enough to avert the challenge
@@ -213,7 +214,7 @@ contract MintingHub {
             delete challenges[_challengeNumber];
         } else {
             // challenge is not averted, update bid
-            if (_bidAmountZCHF < minBid(challenge)) revert BidTooLow(_bidAmountZCHF, minBid(challenge));
+            if (_bidAmountZCHF < _minBid(_bid)) revert BidTooLow(_bidAmountZCHF, _minBid(_bid));
             uint256 earliestEnd = block.timestamp + 30 minutes;
             if (earliestEnd >= challenge.end) {
                 // bump remaining time like ebay does when last minute bids come in
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L287-L296

*Gas Savings for `MintingHub.end`, obtained via protocol's tests: Avg 203 gas* 

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |    -     |    -     |  128303 |    1     |
| After  |    -     |    -     |  128100 |    1     |

```solidity
File: contracts/MintingHub.sol
287:    function returnCollateral(Challenge storage challenge, bool postpone) internal {
288:        if (postpone){
289:            // Postponing helps in case the challenger was blacklisted on the collateral token or otherwise cannot receive it at the moment.
290:            address collateral = address(challenge.position.collateral());
291:            pendingReturns[collateral][challenge.challenger] += challenge.size;
292:            emit PostPonedReturn(collateral, challenge.challenger, challenge.size);
293:        } else {
294:            challenge.position.collateral().transfer(challenge.challenger, challenge.size); // return the challenger's collateral
295:        }
296:    }
```
```diff
diff --git a/contracts/MintingHub.sol b/contracts/MintingHub.sol
index 663b205..5efb447 100644
--- a/contracts/MintingHub.sol
+++ b/contracts/MintingHub.sol
@@ -251,13 +251,23 @@ contract MintingHub {
      */
     function end(uint256 _challengeNumber, bool postponeCollateralReturn) public {
         Challenge storage challenge = challenges[_challengeNumber];
-        require(challenge.challenger != address(0x0));
+        address _challenger = challenge.challenger;
+        require(_challenger != address(0x0));
         require(block.timestamp >= challenge.end, "period has not ended");
         // challenge must have been successful, because otherwise it would have immediately ended on placing the winning bid
-        returnCollateral(challenge, postponeCollateralReturn);
-        // notify the position that will send the collateral to the bidder. If there is no bid, send the collateral to msg.sender
-        address recipient = challenge.bidder == address(0x0) ? msg.sender : challenge.bidder;
-        (address owner, uint256 effectiveBid, uint256 volume, uint256 repayment, uint32 reservePPM) = challenge.position.notifyChallengeSucceeded(recipient, challenge.bid, challenge.size);
+        IPosition _position = challenge.position;
+        address owner;
+        uint256 effectiveBid;
+        uint256 volume;
+        uint256 repayment;
+        uint32 reservePPM;
+        {
+            uint256 _size = challenge.size;
+            returnCollateral(_position, _challenger, _size, postponeCollateralReturn);
+            // notify the position that will send the collateral to the bidder. If there is no bid, send the collateral to msg.sender
+            address recipient = challenge.bidder == address(0x0) ? msg.sender : challenge.bidder;
+            (owner, effectiveBid, volume, repayment, reservePPM) = _position.notifyChallengeSucceeded(recipient, challenge.bid, _size);
+        }
         if (effectiveBid < challenge.bid) {
             // overbid, return excess amount
             IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);
@@ -284,14 +294,14 @@ contract MintingHub {
         IERC20(collateral).transfer(target, amount);
     }

-    function returnCollateral(Challenge storage challenge, bool postpone) internal {
+    function returnCollateral(IPosition _position, address _challenger, uint256 _size, bool postpone) internal {
         if (postpone){
             // Postponing helps in case the challenger was blacklisted on the collateral token or otherwise cannot receive it at the moment.
-            address collateral = address(challenge.position.collateral());
-            pendingReturns[collateral][challenge.challenger] += challenge.size;
-            emit PostPonedReturn(collateral, challenge.challenger, challenge.size);
+            address collateral = address(_position.collateral());
+            pendingReturns[collateral][_challenger] += _size;
+            emit PostPonedReturn(collateral, _challenger, _size);
         } else {
-            challenge.position.collateral().transfer(challenge.challenger, challenge.size); // return the challenger's collateral
+            _position.collateral().transfer(_challenger, _size); // return the challenger's collateral
         }
     }
 }
```

## Cache return value from external call to avoid an unnecessary CALL/STATICCALL
External calls are expensive as they use the `CALL/STATICCALL` opcode (~100 gas). If you are calling the same external function more than once you should cache the return value to avoid an unnecessary `CALL/STATICCALL`.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L268-L276

*Gas Savings for `Position.withdrawCollateral`, obtained via protocol's tests: Avg 998 gas* 

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |    -     |    -     |  73058  |    1     |
| After  |    -     |    -     |  72060  |    1     |

```solidity
File: contracts/Position.sol
268:    function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {
269:        IERC20(collateral).transfer(target, amount);
270:        uint256 balance = collateralBalance();
271:        if (balance < minimumCollateral){
272:            cooldown = expiration;
273:        }
274:        emitUpdate();
275:        return balance;
276:    }
```
```diff
diff --git a/contracts/Position.sol b/contracts/Position.sol
index 3e18534..ea69c6e 100644
--- a/contracts/Position.sol
+++ b/contracts/Position.sol
@@ -271,7 +271,7 @@ contract Position is Ownable, IPosition, MathUtil {
         if (balance < minimumCollateral){
             cooldown = expiration;
         }
-        emitUpdate();
+        emit MintingUpdate(balance, price, minted, limit);
         return balance;
     }
```

## Refactor event to avoid emitting unnecessary storage values that don't change
The `limit` storage variable is only modified in the constructor, the `initializeClone` function, and the `reduceLimitForClone` function. Likewise, the `price` storage variable is only modified in the constructor, the `initilizeClone` function, and the `adjustPrice` function. Instead of using the `MintingUpdate` event (which emits the `limit` and `price` storage variables) in functions that do not modify `limit` and `price`, we can create a new event that opts out of emitting those excessive storage variables. 

**Note: Only `repay` and `withdrawCollateral` undergo this optimization since those are the only functions that are benchmarked in the tests. This optimization can also be done for other functions that call `emitUpdate` and do not modify the `limit` and `price` storage variables**.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L232-L236

*Gas Savings for `Position.repay`, obtained via protocol's tests: Avg 4297 gas* 

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |    -     |    -     |  87902  |    2     |
| After  |    -     |    -     |  83605  |    2     |

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L268-L276

*Gas Savings for `Position.withdrawCollateral`, obtained via protocol's tests: Avg 2775 gas* 

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |    -     |    -     |  73058  |    1     |
| After  |    -     |    -     |  70283  |    1     |

```solidity
File: contracts/Position.sol
232:    function repayInternal(uint256 burnable) internal {
233:        uint256 actuallyBurned = IFrankencoin(zchf).burnWithReserve(burnable, reserveContribution);
234:        notifyRepaidInternal(actuallyBurned);
235:        emitUpdate();
236:    }

268:    function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {
269:        IERC20(collateral).transfer(target, amount);
270:        uint256 balance = collateralBalance();
271:        if (balance < minimumCollateral){
272:            cooldown = expiration;
273:        }
274:        emitUpdate();
275:        return balance;
276:    }
```
```diff
diff --git a/contracts/Position.sol b/contracts/Position.sol
index 3e18534..7e5f7f9 100644
--- a/contracts/Position.sol
+++ b/contracts/Position.sol
@@ -40,6 +40,7 @@ contract Position is Ownable, IPosition, MathUtil {

     event PositionOpened(address indexed owner, address original, address zchf, address collateral, uint256 price);
     event MintingUpdate(uint256 collateral, uint256 price, uint256 minted, uint256 limit);
+    event Update(uint256 collateral, uint256 minted);
     event PositionDenied(address indexed sender, string message); // emitted if closed by governance

     error InsufficientCollateral();
@@ -232,7 +233,7 @@ contract Position is Ownable, IPosition, MathUtil {
     function repayInternal(uint256 burnable) internal {
         uint256 actuallyBurned = IFrankencoin(zchf).burnWithReserve(burnable, reserveContribution);
         notifyRepaidInternal(actuallyBurned);
-        emitUpdate();
+        emit Update(collateralBalance(), minted);
     }

     error RepaidTooMuch(uint256 excess);
@@ -271,7 +272,7 @@ contract Position is Ownable, IPosition, MathUtil {
         if (balance < minimumCollateral){
             cooldown = expiration;
         }
-        emitUpdate();
+        emit Update(collateralBalance(), minted);
         return balance;
     }
```

## Use `unchecked` for expressions that can not underflow due to previous `if`/`require` statement
Wrapping expressions in unchecked blocks will tell the solidity compiler to exclude the extra opcodes needed to check for underflows.

Total Instances: `2`

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L240-L243

*Gas Savings for `Position.repay`, obtained via protocol's tests: Avg 66 gas* 

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |    -     |    -     |  87902  |    2     |
| After  |    -     |    -     |  87836  |    2     |

```solidity
File: contracts/Position.sol
240:    function notifyRepaidInternal(uint256 amount) internal {
241:        if (amount > minted) revert RepaidTooMuch(amount - minted);
242:        minted -= amount;
243:    }
```
```diff
diff --git a/contracts/Position.sol b/contracts/Position.sol
index 3e18534..b251af4 100644
--- a/contracts/Position.sol
+++ b/contracts/Position.sol
@@ -239,7 +239,9 @@ contract Position is Ownable, IPosition, MathUtil {

     function notifyRepaidInternal(uint256 amount) internal {
         if (amount > minted) revert RepaidTooMuch(amount - minted);
-        minted -= amount;
+        unchecked {
+            minted -= amount;
+        }
     }
```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L261-L271

*Gas Savings for `MintingHub.end`, obtained via protocol's tests: Avg 80 gas* 

|        |    Min   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |    -     |    -     |  128303 |    1     |
| After  |    -     |    -     |  128223 |    1     |

```solidity
File: contracts/MintingHub.sol
261:        if (effectiveBid < challenge.bid) {
262:            // overbid, return excess amount
263:            IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);
264:        }
265:        uint256 reward = (volume * CHALLENGER_REWARD) / 1000_000;
266:        uint256 fundsNeeded = reward + repayment;
267:        if (effectiveBid > fundsNeeded){
268:            zchf.transfer(owner, effectiveBid - fundsNeeded);
269:        } else if (effectiveBid < fundsNeeded){
270:            zchf.notifyLoss(fundsNeeded - effectiveBid); // ensure we have enough to pay everything
271:        }
```
```diff
diff --git a/contracts/MintingHub.sol b/contracts/MintingHub.sol
index 663b205..6854a9e 100644
--- a/contracts/MintingHub.sol
+++ b/contracts/MintingHub.sol
@@ -260,14 +260,20 @@ contract MintingHub {
         (address owner, uint256 effectiveBid, uint256 volume, uint256 repayment, uint32 reservePPM) = challenge.position.notifyChallengeSucceeded(recipient, challenge.bid, challenge.size);
         if (effectiveBid < challenge.bid) {
             // overbid, return excess amount
-            IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);
+            unchecked {
+                IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);
+            }
         }
         uint256 reward = (volume * CHALLENGER_REWARD) / 1000_000;
         uint256 fundsNeeded = reward + repayment;
         if (effectiveBid > fundsNeeded){
-            zchf.transfer(owner, effectiveBid - fundsNeeded);
+            unchecked {
+                zchf.transfer(owner, effectiveBid - fundsNeeded);
+            }
         } else if (effectiveBid < fundsNeeded){
-            zchf.notifyLoss(fundsNeeded - effectiveBid); // ensure we have enough to pay everything
+            unchecked {
+                zchf.notifyLoss(fundsNeeded - effectiveBid); // ensure we have enough to pay everything
+            }
         }
```

## `GasReport` output, with all optimizations applied

```js
·-------------------------------------------------|---------------------------|-------------|-----------------------------·
|              Solc version: 0.8.13               ·  Optimizer enabled: true  ·  Runs: 200  ·  Block limit: 30000000 gas  │
··················································|···························|·············|······························
|  Methods                                                                                                                │
·····················|····························|·············|·············|·············|···············|··············
|  Contract          ·  Method                    ·  Min        ·  Max        ·  Avg        ·  # calls      ·  usd (avg)  │
·····················|····························|·············|·············|·············|···············|··············
|  Equity            ·  delegateVoteTo            ·      45534  ·      45546  ·      45540  ·            2  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Equity            ·  redeem                    ·          -  ·          -  ·      66819  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Equity            ·  transfer                  ·          -  ·          -  ·      85527  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Frankencoin       ·  approve                   ·          -  ·          -  ·      46228  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Frankencoin       ·  burn                      ·          -  ·          -  ·      33770  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Frankencoin       ·  denyMinter                ·          -  ·          -  ·      39074  ·            2  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Frankencoin       ·  mint                      ·          -  ·          -  ·     116933  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Frankencoin       ·  suggestMinter             ·      56607  ·      77697  ·      62144  ·            8  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Frankencoin       ·  transfer                  ·      34528  ·      51616  ·      40224  ·            3  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Frankencoin       ·  transferAndCall           ·      65972  ·     156127  ·     113385  ·            4  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHub        ·  bid                       ·      73363  ·     120804  ·      97084  ·            4  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHub        ·  clonePosition             ·          -  ·          -  ·     274904  ·            2  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHub        ·  end                       ·          -  ·          -  ·     121752  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHub        ·  launchChallenge           ·          -  ·          -  ·     210751  ·            2  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHub        ·  openPosition              ·          -  ·          -  ·    1737410  ·            2  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  bidNearEndOfChallenge     ·          -  ·          -  ·     140715  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  challengeExpiredPosition  ·          -  ·          -  ·     272504  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  endChallenges             ·          -  ·          -  ·     469531  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  endLastChallenge          ·          -  ·          -  ·     119344  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  initiateAndDenyPosition   ·          -  ·          -  ·    1831819  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  initiateEquity            ·          -  ·          -  ·     365145  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  initiatePosition          ·          -  ·          -  ·    1849030  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  letAliceMint              ·          -  ·          -  ·     311827  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  letBobChallenge           ·          -  ·          -  ·     705114  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  restructure               ·          -  ·          -  ·     183340  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  testExcessiveChallenge    ·          -  ·          -  ·     290328  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  MintingHubTest    ·  testWithdraw              ·          -  ·          -  ·     169588  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Position          ·  repay                     ·      82439  ·      83148  ·      82794  ·            2  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  Position          ·  withdrawCollateral        ·          -  ·          -  ·      69071  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  StablecoinBridge  ·  burn                      ·          -  ·          -  ·      72239  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  StablecoinBridge  ·  burn                      ·          -  ·          -  ·      55545  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  StablecoinBridge  ·  mint                      ·      75906  ·     110106  ·      97904  ·            6  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  TestToken         ·  approve                   ·      29093  ·      46205  ·      44878  ·           13  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  TestToken         ·  mint                      ·      51294  ·      68394  ·      64118  ·            8  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  TestToken         ·  transfer                  ·          -  ·          -  ·      51614  ·            2  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
|  TestToken         ·  transferAndCall           ·          -  ·          -  ·      52939  ·            1  ·          -  │
·····················|····························|·············|·············|·············|···············|··············
```