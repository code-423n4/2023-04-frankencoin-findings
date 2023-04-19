## Summary
### Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|L-01|Danger "while" loop| 1 |
|L-02|Function Calls in Loop Could Lead to Denial of Service due to Array length not being checked| 1 |
|L-03|Add `salt` when creating `new Position` with `createNewPosition` function | 1 |
|L-04|`votes()` functions with the same name cause confusion|1|
|L-05|Array size is large| 1 |
|L-06|Use OpenZeppelin contracts by importing the latest versions instead of direct use| 1 |
|L-07|Project Upgrade and Stop Scenario should be | All Contracts |
|L-08|Insufficient coverage | All Contracts |
|L-09|Add a timelock to `adjustPrice()` function| 1 |
|L-10|Consider using OpenZeppelin's SafeCast library to prevent unexpected overflows when casting from uint256| 1 |
|NC-11 |Constants on the left are better|4|
|NC-12 |Assembly Codes Specific – Should Have Comments| 1|
|NC-13 |For functions, follow Solidity standard naming conventions (internal function style rule)| 23|
|NC-14 |Use SMTChecker|All Contracts|
|NC-15 |Use `uint256` instead `uint`| 2 |
|NC-16 |Avoid _shadowing_ `inherited state variables`| 1 |
|NC-17 |Repeated validation logic can be converted into a function modifier| 3 |
|NC-18 |Showing the actual values of numbers in NatSpec comments makes checking and reading code easier| 4 |
|NC-19 |Use a single file for all system-wide constants| 11 |
|NC-20 |Use of `_beforeTokenTransfer` is unnecessary, can be removed| 3|
|NC-21|Missing Event for  initialize| 5 |

Total 21 issues


## [L-01] Danger "while" loop

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L27

```solidity

contracts/MathUtil.sol:
  17       */
  18:     function _cubicRoot(uint256 _v) internal pure returns (uint256) {
  19:         uint256 x = ONE_DEC18;
  20:         uint256 xOld;
  21:         bool cond;
  22:         do {
  23:             xOld = x;
  24:             uint256 powX3 = _mulD18(_mulD18(x, x), x);
  25:             x = _mulD18(x, _divD18( (powX3 + 2 * _v) , (2 * powX3 + _v)));
  26:             cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
  27:         } while ( cond ); @audit
  28:         return x;
  29:     }

```
## Impact
` MathUtil.sol#L27` has a while loop on this line .
When using `while` loops, the Solidity compiler is aware that a condition needs to be checked first, before executing the code inside the loop.

Firstly, the conditional part is evaluated

a) if the condition evaluates to true , the instructions inside the loop (defined inside the brackets { ... }) get executed.

b) once the code reaches the end of the loop body (= the closing curly brace } , it will jump back up to the conditional part).

c) the code inside the while loop body will keep executing over and over again until the conditional part turns false.

For loops are more associated with iterations. While loop instead are more associated with repetitive tasks. Such tasks could potentially be infinite if the while loop body never affects the condition initially checked.
Therefore, they should be used with care.


At very small intervals, a large number of loops can be accessed by someone attempting a DOS attack.


## Tools Used
Manual code review

## Recommended Mitigation Steps
Determine a reasonable minimum range , according to the network used, a for loop number can be determined according to the block gas limit, for example the number 60 is specified here.

```solidity
function _cubicRoot(uint256 _v) internal pure returns (uint256) {
    uint256 x = ONE_DEC18;
    uint256 powX3;
    for (uint256 i = 0; i < 60; i++) {
        powX3 = _mulD18(_mulD18(x, x), x);
        x = _mulD18(x, _divD18( (powX3 + 2 * _v) , (2 * powX3 + _v)));
    }
    return x;
}

```

### [L-02] Function Calls in Loop Could Lead to Denial of Service due to Array length not being checked


```diff

contracts/Equity.sol:

+ maxArrayLength = 50;

  190:     function votes(address sender, address[] calldata helpers) public view returns (uint256) {
  191:         uint256 _votes = votes(sender);
+      if helpers.length > maxArrayLength) {
+          revert maxArrayLengt();
+       } 
  192:         for (uint i=0; i<helpers.length; i++){
  193:             address current = helpers[i];
  194:             require(current != sender);
  195:             require(canVoteFor(sender, current));
  196:             for (uint j=i+1; j<helpers.length; j++){
  197:                 require(current != helpers[j]); // ensure helper unique
  198:             }
  199:             _votes += votes(current);
  200:         }
  201:         return _votes;
  202:     }



```

**Recommendation:**
Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to gas limitations or failed transactions. Consider bounding the loop length if the array is expected to be growing and/or handling a huge list of elements to avoid unnecessary gas wastage and denial of service.


### [L-03] Add `salt` when creating `new Position` with `createNewPosition` function

Using the salt parameter when creating a contract instance is an important security measure that can help protect against replay attacks and reorg vulnerabilities.


```solidity

contracts/PositionFactory.sol:
  12       */
  13:     function createNewPosition(address _owner, address _zchf, address _collateral, uint256 _minCollateral, 
  14:         uint256 _initialLimit, uint256 initPeriod, uint256 _duration, uint256 _challengePeriod, 
  15:         uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reserve) 
  16:         external returns (address) {
  17:         return address(new Position(_owner, msg.sender, _zchf, _collateral, 
  18:             _minCollateral, _initialLimit, initPeriod, _duration, 
  19:             _challengePeriod, _mintingFeePPM, _liqPrice, _reserve));
  20:     }


```

**Recommendation:**

The salt value can be generated using a unique identifier such as a nonce and combined with other constructor arguments to create a unique identifier for the new contract.

Using a `salt` parameter with the keyword ensures that each new contract instance has a unique address

When using the `new` key, the risk is low as msg.sender is used, but again purely important due to the possibility of msg.sender attack


add this code instead above original code ;

```solidity

function createNewPosition(address _owner, address _zchf, address _collateral, uint256 _minCollateral, 
    uint256 _initialLimit, uint256 initPeriod, uint256 _duration, uint256 _challengePeriod, 
    uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reserve, uint256 _salt)  external returns (address) {
    bytes32 salt = keccak256(abi.encodePacked(_owner, _zchf, _collateral, _minCollateral, 
        _initialLimit, initPeriod, _duration, _challengePeriod, _mintingFeePPM, _liqPrice, _reserve, _salt));
        
    return address(new Position{salt: salt}(_owner, msg.sender, _zchf, _collateral, 
        _minCollateral, _initialLimit, initPeriod, _duration, 
        _challengePeriod, _mintingFeePPM, _liqPrice, _reserve));
}

```

### [L-04] `votes()` functions with the same name cause confusion

`votes()` functions with the same name cause confusion

Project has two same name `votes()` function with different type and amount parameters , this can be cause confusion although different parameters ,
Use to different function name is best practice

```solidity

contracts/Equity.sol:

  179:     function votes(address holder) public view returns (uint256) {
  180:         return balanceOf(holder) * (anchorTime() - voteAnchor[holder]);
  181:     }


  190:     function votes(address sender, address[] calldata helpers) public view returns (uint256) {
  191:         uint256 _votes = votes(sender);
  192:         for (uint i=0; i<helpers.length; i++){
  193:             address current = helpers[i];
  194:             require(current != sender);
  195:             require(canVoteFor(sender, current));
  196:             for (uint j=i+1; j<helpers.length; j++){
  197:                 require(current != helpers[j]); // ensure helper unique
  198:             }
  199:             _votes += votes(current);
  200:         }
  201:         return _votes;
  202:     }



```


**Recommendation:**
Use different function name by style of doing it work


### [L-05] Array size is large

``Challenge[] public challenges`` is used to list of open challenges. Users can ``push`` to strucks in ``Challenges[]`` from ``launchChallenge()`` and ``splitChallenge()`` functions. When ``Challenge[] public challenges`` array size is large, some functions ``revert`` due to running DOS


Therefore, the size of the **challenges** dynamic array should be checked and the upper bound should be determined. When the upper limit is reached, it should be ensured that no additional push is made.

Thus, the problem of running out of gas is prevented.

```solidity
contracts\MintingHub.sol:

  31:     Challenge[] public challenges; // list of open challenges

+         uint256 constant private MAX_LENGTH_CHALLENGE = 150; // max number of challenges 

  140:     function launchChallenge(address _positionAddr, uint256 _collateralAmount) external validPos(_positionAddr) returns (uint256) {
+ 143:         uint256 pos = challenges.length;
+              require(pos < MAX_LENGTH_CHALLENGE, "max number of challenges")
  141:         IPosition position = IPosition(_positionAddr);
  142:         IERC20(position.collateral()).transferFrom(msg.sender, address(this), _collateralAmount);
- 143:         uint256 pos = challenges.length;
  144:         challenges.push(Challenge(msg.sender, position, _collateralAmount, block.timestamp + position.challengePeriod(), address(0x0), 0));
  145:         position.notifyChallengeStarted(_collateralAmount);
  146:         emit ChallengeStarted(msg.sender, address(position), _collateralAmount, pos);
  147:         return pos;
  148:     }

```

```solidity
  156:     function splitChallenge(uint256 _challengeNumber, uint256 splitOffAmount) external returns (uint256) {
+  174:         uint256 pos = challenges.length;
+              require(pos < MAX_LENGTH_CHALLENGE, "max number of challenges")
  157:         Challenge storage challenge = challenges[_challengeNumber];
  158:         require(challenge.challenger != address(0x0));
  159:         Challenge memory copy = Challenge(
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
  172:         require(copy.size >= min);
  173: 
- 174:         uint256 pos = challenges.length;
  175:         challenges.push(copy);
  176:         emit ChallengeStarted(challenge.challenger, address(challenge.position), challenge.size, _challengeNumber);
  177:         emit ChallengeStarted(copy.challenger, address(copy.position), copy.size, pos);
  178:         return pos;
  179:     }

```

### [L-06] Use OpenZeppelin contracts by importing the latest versions instead of direct use


```solidity
contracts/ERC20.sol:
   2: // Copied and adjusted from OpenZeppelin
  53:     // Copied from https://github.com/OpenZeppelin/openzeppelin-contracts/pull/4139/files#diff-fa792f7d08644eebc519dac2c29b00a54afc4c6a76b9ef3bba56c8401fe674f6

contracts/Ownable.sol:
  3: // From https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable.sol



```

Use OpenZeppelin contracts by importing the latest versions for vulnerabilities;

https://github.com/OpenZeppelin/openzeppelin-contracts/releases/


### Recommended Mitigation Steps

```js
npm install @openzeppelin/contracts

```

```solidity
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";


```


### [L-07] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

### [L-08] Insufficient coverage

**Description:**
The test coverage rate of the project is ~98%. Testing all functions is best practice in terms of security criteria.
Due to its capacity, test coverage is expected to be 100%.

If it is not 100% for certain reasons, the reason should be stated in the documents.

Of course, this ratio is a good ratio and can be tolerated, but it should be noted that many of the safety problems come from these small tolerances.

```js
README.md:
  130: - What is the overall line coverage percentage provided by your tests?: 98%
```


### [L-09] Add a timelock to `adjustPrice()` function


It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).
Consider adding a timelock to:


The adjustPrice() function has a potential vulnerability as it allows the position owner to immediately adjust the liquidation price without any time delay. 

This could lead to a situation where the position becomes under-collateralized and vulnerable to liquidation if the owner lowers the price without enough collateral. 

```solidity

contracts/Position.sol:
  154:     /**
  155:      * Allows the position owner to adjust the liquidation price as long as there is no pending challenge.
  156:      * Lowering the liquidation price can be done with immediate effect, given that there is enough collateral.
  157:      * Increasing the liquidation price triggers a cooldown period of 3 days, during which minting is suspended.
  158:      */
  159:     function adjustPrice(uint256 newPrice) public onlyOwner noChallenge {
  160:         if (newPrice > price) {
  161:             restrictMinting(3 days);
  162:         } else {
  163:             checkCollateral(collateralBalance(), newPrice);
  164:         }
  165:         price = newPrice;
  166:         emitUpdate();
  167:     }


```

### [L-10] Consider using OpenZeppelin's SafeCast library to prevent unexpected overflows when casting from uint256


```solidity


contracts/Equity.sol:
  146:         totalVotesAtAnchor = uint192(totalVotes() - roundingLoss - lostVotes);

```
In the Equity contract has  the `spend`, ` roundingLoss `, `  lostVotes`  variables are type uint256, than ,in the function, they are downcasted to uint192


Recommended Mitigation Steps:
Consider using OpenZeppelin's SafeCast library to prevent unexpected overflows when casting from uint256.

### [N-11] Constants on the left are better

If you use the constant first you support structures that veil programming errors. And one should only produce code either to add functionality, fix an programming error or trying to support structures to avoid programming errors (like design patterns). 

https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html

```solidity
4 results - 2 files

contracts/Equity.sol:
  145:         uint256 lostVotes = from == address(0x0) ? 0 : (anchorTime() - voteAnchor[from]) * amount;
  228:         } else if (owner == address(0x0)){

contracts/MintingHub.sol:
  116:         require(zchf.isPosition(position) == address(this), "not our pos");
  259:         address recipient = challenge.bidder == address(0x0) ? msg.sender : challenge.bidder;


```



### [N-12] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does


This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Aseembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

```solidity
contracts/PositionFactory.sol:
  37:     function createClone(address target) internal returns (address result) {
  38:         bytes20 targetBytes = bytes20(target);
  39:         assembly {
  40:             let clone := mload(0x40)
  41:             mstore(clone, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
  42:             mstore(add(clone, 0x14), targetBytes)
  43:             mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
  44:             result := create(0, clone, 0x37)
  45:         }
  46:     }

```


### [N-13] For functions, follow Solidity standard naming conventions (internal function style rule)

**Context:**
```solidity

23 results - 9 files

contracts/Equity.sol:
  144:     function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {
  157:     function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){
  172:     function anchorTime() internal view returns (uint64){
  225:     function canVoteFor(address delegate, address owner) internal view returns (bool) {
  266:     function calculateSharesInternal(uint256 capitalBefore, uint256 investment) internal view returns (uint256) {

contracts/ERC20.sol:
  97:     function allowanceInternal(address owner, address spender) internal view virtual returns (uint256) {

contracts/Frankencoin.sol:
  102:    function allowanceInternal(address owner, address spender) internal view override returns (uint256) {

contracts/MathUtil.sol:
  10:     uint256 internal constant ONE_DEC18 = 10**18;

contracts/MintingHub.sol:
  188:     function minBid(Challenge storage challenge) internal view returns (uint256) {
  287:     function returnCollateral(Challenge storage challenge, bool postpone) internal {

contracts/Ownable.sol:
  39:     function setOwner(address newOwner) internal {
  45:     function requireOwner(address sender) internal view {

contracts/Position.sol:
  169:     function collateralBalance() internal view returns (uint256){
  193:     function mintInternal(address target, uint256 amount, uint256 collateral_) internal {
  202:     function restrictMinting(uint256 period) internal {
  232:     function repayInternal(uint256 burnable) internal {
  240:     function notifyRepaidInternal(uint256 amount) internal {
  268:     function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {
  282:     function checkCollateral(uint256 collateralReserve, uint256 atPrice) internal view {
  286:     function emitUpdate() internal {

contracts/PositionFactory.sol:
  37:     function createClone(address target) internal returns (address result) {

contracts/StablecoinBridge.sol:
  49:     function mintInternal(address target, uint256 amount) internal {
  67:     function burnInternal(address zchfHolder, address target, uint256 amount) internal {



```


**Description:**
The above codes don't follow Solidity's standard naming convention,

internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)



### [N-14] Use SMTChecker

The *highest* tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19


### [N-15] Use `uint256` instead `uint`

```solidity

contracts/Equity.sol:
  91:     event Trade(address who, int amount, uint totPrice, uint newprice); 

```
Project use uint and uint256
 
Number of uses:
uint  = 2 results
uint256 = 188 results

Some developers prefer to use `uint256` because it is consistent with other uint data types, which also specify their size, and also because making the size of the data explicit reminds the developer and the reader how much data they've got to play with, which may help prevent or detect bugs.



## [N-16] Avoid _shadowing_ `inherited state variables`

**Context:**
```solidity

contracts/Ownable.sol:
  21:     address public owner;


contracts/Position.sol:
  76:     function initializeClone(address owner, uint256 _price, uint256 _limit, uint256 _coll, uint256 _mint) external onlyHub {
  77:         if(_coll < minimumCollateral) revert InsufficientCollateral();
  78:         setOwner(owner);
  79:         
  80:         price = _mint * ONE_DEC18 / _coll;
  81:         if (price > _price) revert InsufficientCollateral();
  82:         limit = _limit;
  83:         mintInternal(owner, _mint, _coll);
  84: 
  85:         emit PositionOpened(owner, original, address(zchf), address(collateral), _price);
  86:     }
```


`owner` is shadowed, 

The local variable name `owner` in the ` Position.sol ` hase the same name and create shadowing

**Recommendation:**
Avoid using variables with the same name, including inherited in the same contract, if used, it must be specified in the NatSpec comments.



### [N-17] Repeated validation logic can be converted into a function modifier

If a query or logic is repeated over many lines, using a modifier improves the readability and reusability of the code
(!= address(0))

```solidity

3 results - 2 files

contracts/ERC20.sol:
  151      function _transfer(address sender, address recipient, uint256 amount) internal virtual {
  152:         require(recipient != address(0));

  179      function _mint(address recipient, uint256 amount) internal virtual {
  180:         require(recipient != address(0));

contracts/ERC20PermitLight.sol:
  55  
  56:             require(recoveredAddress != address(0) && recoveredAddress == owner, "INVALID_SIGNER");

```



### [N-18] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier


```diff

4 results - 2 files

contracts/MintingHub.sol:
- 64:             7 days, _expirationSeconds, _challengeSeconds, _mintingFeePPM, _liqPrice, _reservePPM);
+	  7 days, _expirationSeconds, _challengeSeconds, _mintingFeePPM, _liqPrice, _reservePPM); // 7 days ( 7*24*60*60 ) = 604_800

contracts/Position.sol:
-  53:         require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values
+                require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values // 3days ( 3*24*60*60 ) = 259_200

- 161:             restrictMinting(3 days); 
+	      restrictMinting(3 days);  // 3days ( 3*24*60*60 ) = 259_200

- 312:             restrictMinting(1 days);
+	      restrictMinting(1 days); // 1days ( 1*24*60*60 ) = 86_400

```

### [N-19] Use a single file for all system-wide constants

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values)

  This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.

constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

```solidity
11 results - 5 files

contracts/Equity.sol:
  39:     uint32 public constant VALUATION_FACTOR = 3;
  41:     uint256 private constant MINIMUM_EQUITY = 1000 * ONE_DEC18;
  46:     uint32 private constant QUORUM = 300;
  51:     uint8 private constant BLOCK_TIME_RESOLUTION_BITS = 24;
  59:     uint256 public constant MIN_HOLDING_DURATION = 90*7200 << BLOCK_TIME_RESOLUTION_BITS; // Set to 5 for local testing

contracts/ERC20.sol:
  47:     uint256 internal constant INFINITY = (1 << 255);

contracts/Frankencoin.sol:
  25:    uint256 public constant MIN_FEE = 1000 * (10**18);

contracts/MathUtil.sol:
  10:     uint256 internal constant ONE_DEC18 = 10**18;
  11:     uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01

contracts/MintingHub.sol:
  20:     uint256 public constant OPENING_FEE = 1000 * 10**18;
  26:     uint32 public constant CHALLENGER_REWARD = 20000; // 2%
```


### [N-20] Use of `_beforeTokenTransfer` is unnecessary, can be removed

 `_mint()` , `_burn()` ,  `_transfer()` functions are used to `_beforeTokenTransfer` hook but this hook is unneccassary 
other hand   `_afterTokenTransfer` hook  isn,’t used by  ERC20.sol  and  this can be non consistently


```solidity
contracts/ERC20.sol:
  178:      */
  179:     function _mint(address recipient, uint256 amount) internal virtual {
  180:         require(recipient != address(0));
  181: 
  182:         _beforeTokenTransfer(address(0), recipient, amount);
  183: 
  184:         _totalSupply += amount;
  185:         _balances[recipient] += amount;
  186:         emit Transfer(address(0), recipient, amount);
  187:     }


 240:     function _beforeTokenTransfer(address from, address to, uint256 amount) virtual internal {}

```

### [N-21] Missing Event for  initialize

**Context:**
```solidity

contracts/Equity.sol:
  92  
  93:     constructor(Frankencoin zchf_) ERC20(18) {
  94:         zchf = zchf_;
  95:     }

contracts/ERC20.sol:
  59:     constructor(uint8 _decimals) {
  60:         decimals = _decimals;
  61:     }

contracts/Frankencoin.sol:
  59:    constructor(uint256 _minApplicationPeriod) ERC20(18){
  60:       MIN_APPLICATION_PERIOD = _minApplicationPeriod;
  61:       reserve = new Equity(this);
  62:    }

contracts/MintingHub.sol:
  54:     constructor(address _zchf, address factory) {
  55:         zchf = IFrankencoin(_zchf);
  56:         POSITION_FACTORY = IPositionFactory(factory);
  57:     }


contracts/StablecoinBridge.sol:
  26:     constructor(address other, address zchfAddress, uint256 limit_){
  27:         chf = IERC20(other);
  28:         zchf = IFrankencoin(zchfAddress);
  29:         horizon = block.timestamp + 52 weeks;
  30:         limit = limit_;
  31:     }

```

**Description:**
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
Issuing event-emit during initialization is a detail that many projects skip

**Recommendation:**
Add Event-Emit
