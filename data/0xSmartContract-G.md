### Gas Optimizations List
| Number | Optimization Details | Context |
|:--:|:-------| :-----:|
| [G-01] |When ``Challenge[] public challenges`` struct is large array size some functions ``revert`` due to running out of gas| 1 |
| [G-02] |State variables should be cached in stack variables rather than re-reading them from storagee| 2 |
| [G-03] |In permit() function use ``_allowances[owner][spender] = value;`` instead of ``_approve()`` |1 |
| [G-04] |Gas overflow during iteration (DoS) | 2 |
| [G-05] |Missing `zero-address` check in `constructor` |4|
| [G-06] |Avoid contract existence checks by using solidity version 0.8.10 or later | 36 |
| [G-07] |if () / require() statements that check input arguments should be at the top of the function| 1 |
| [G-08] | Use nested if and, avoid multiple check combinations| 3 |
| [G-09] | Ternary operation is cheaper than if-else statement | 2 |
| [G-10] |Upgrade Solidity's optimizer |  |

Total 10 issues


### [G-01] When ``Challenge[] public challenges`` struct is large array size some functions ``revert`` due to running out of gas

``Challenge[] public challenges`` is used to list of open challenges. Users can ``push`` to struct in ``Challenges[]`` from ``launchChallenge()`` and ``splitChallenge()`` functions.

Therefore, the size of the **challenges** dynamic array should be checked and the upper bound should be determined. When the upper limit is reached, it should be ensured that no additional push is made.

Thus, the problem of running out of gas is prevented.


```diff
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
  

  156:     function splitChallenge(uint256 _challengeNumber, uint256 splitOffAmount) external returns (uint256) {
+ 174:         uint256 pos = challenges.length;
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

### [G-02] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.

2 results - 1 files:
```solidity
contracts\Frankencoin.sol:

  280:    function notifyLoss(uint256 _amount) override external minterOnly {
  281:       uint256 reserveLeft = balanceOf(address(reserve));
  282:       if (reserveLeft >= _amount){
  283:          _transfer(address(reserve), msg.sender, _amount);
  284:       } else {
  285:          _transfer(address(reserve), msg.sender, reserveLeft);
  286:          _mint(msg.sender, _amount - reserveLeft);
  287:       }
  288:    }

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L280-L288

```diff
contracts\Frankencoin.sol:
  280:    function notifyLoss(uint256 _amount) override external minterOnly {
+            address _reserve = address(reserve);
- 281:       uint256 reserveLeft = balanceOf(address(reserve));
+ 281:       uint256 reserveLeft = balanceOf(address(_reserve));
  282:       if (reserveLeft >= _amount){
- 283:          _transfer(address(reserve), msg.sender, _amount);
+ 283:          _transfer(address(_reserve), msg.sender, _amount);
  284:       } else {
- 285:          _transfer(address(reserve), msg.sender, reserveLeft);
+ 285:          _transfer(address(_reserve), msg.sender, reserveLeft);
  286:          _mint(msg.sender, _amount - reserveLeft);
  287:       }
  288:    }

```

```solidity
contracts\Position.sol:
  132:     function adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) public onlyOwner {
  133:         if (newPrice != price){
  134:             adjustPrice(newPrice);
  135:         }
  136:         uint256 colbal = collateralBalance();
  137:         if (newCollateral > colbal){
  138:             collateral.transferFrom(msg.sender, address(this), newCollateral - colbal);
  139:         }
  140:         // Must be called after collateral deposit, but before withdrawal
  141:         if (newMinted < minted){
  142:             zchf.burnFrom(msg.sender, minted - newMinted, reserveContribution);
  143:             minted = newMinted;
  144:         }
  145:         if (newCollateral < colbal){
  146:             withdrawCollateral(msg.sender, colbal - newCollateral);
  147:         }
  148:         // Must be called after collateral withdrawal
  149:         if (newMinted > minted){
  150:             mint(msg.sender, newMinted - minted);
  151:         }
  152:     }

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L132-L152


```diff
contracts\Position.sol:
  132:     function adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) public onlyOwner {
  133:         if (newPrice != price){
  134:             adjustPrice(newPrice);
  135:         }
  136:         uint256 colbal = collateralBalance();
  137:         if (newCollateral > colbal){
  138:             collateral.transferFrom(msg.sender, address(this), newCollateral - colbal);
  139:         }
  +            uint256 _minted = minted
  140:         // Must be called after collateral deposit, but before withdrawal
- 141:         if (newMinted < minted){
+ 141:         if (newMinted < _minted){
- 142:             zchf.burnFrom(msg.sender, minted - newMinted, reserveContribution);
+ 142:             zchf.burnFrom(msg.sender, _minted - newMinted, reserveContribution);
  143:             minted = newMinted;
  144:         }
  145:         if (newCollateral < colbal){
  146:             withdrawCollateral(msg.sender, colbal - newCollateral);
  147:         }
  148:         // Must be called after collateral withdrawal
- 149:         if (newMinted > minted){
+ 149:         if (newMinted > _minted){
  150:             mint(msg.sender, newMinted - _minted);
  151:         }
  152:     }

```

### [G-03] In permit() function use ``_allowances[owner][spender] = value;`` instead of ``_approve()``


```diff
contracts\ERC20PermitLight.sol:

  21:     function permit(
  22:         address owner,
  23:         address spender,
  24:         uint256 value,
  25:         uint256 deadline,
  26:         uint8 v,
  27:         bytes32 r,
  28:         bytes32 s
  29:     ) public {
  30:         require(deadline >= block.timestamp, "PERMIT_DEADLINE_EXPIRED");
  31: 
  32:         unchecked { // unchecked to save a little gas with the nonce increment...
  33:             address recoveredAddress = ecrecover(
  34:                 keccak256(
  35:                     abi.encodePacked(
  36:                         "\x19\x01",
  37:                         DOMAIN_SEPARATOR(),
  38:                         keccak256(
  39:                             abi.encode(
  40:                                 // keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),
  41:                                 bytes32(0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9),
  42:                                 owner,
  43:                                 spender,
  44:                                 value,
  45:                                 nonces[owner]++,
  46:                                 deadline
  47:                             )
  48:                         )
  49:                     )
  50:                 ),
  51:                 v,
  52:                 r,
  53:                 s
  54:             );
  55: 
  56:             require(recoveredAddress != address(0) && recoveredAddress == owner, "INVALID_SIGNER");
- 57:             _approve(recoveredAddress, spender, value);
+                 _allowances[owner][spender] = value;
  58:         }

+             emit Approval(owner, spender, value);  
  59:     }

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L21-L59


### [G-04] Gas overflow during iteration (DoS)

Each iteration of the cycle requires a gas flow. A moment may come when more gas is required than it is allocated to record one block. In this case, all iterations of the loop will fail.

2 results - 1 file:

```diff
contracts/Equity.sol:

+          error maxArrayLengt();
+          uint256 private constant MAX_ARRAY_LENGTH = 50;

  190:     function votes(address sender, address[] calldata helpers) public view returns (uint256) {
  191:         uint256 _votes = votes(sender);
+              if helpers.length > maxArrayLength) {
+                  revert maxArrayLengt();
+              } 
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
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L190-L202



```diff
contracts\Equity.sol:

+          error maxArrayLengt();
+          uint256 private constant MAX_ARRAY_LENGTH = 50;

  309:     function restructureCapTable(address[] calldata helpers, address[] calldata addressesToWipe) public {
  310:         require(zchf.equity() < MINIMUM_EQUITY);
  311:         checkQualified(msg.sender, helpers);

+              if addressesToWipe.length > maxArrayLength) {
+                  revert maxArrayLengt();
+              } 
  312:         for (uint256 i = 0; i<addressesToWipe.length; i++){
  313:             address current = addressesToWipe[0];
  314:             _burn(current, balanceOf(current));
  315:         }
  316:   }

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L309-L316


###  [G-05] Missing `zero-address` check in `constructor`

While the following critical fee values and address variables are assigned in the constructor, there is no zero value control. Initializing these variables with a possible value of "0" means that the contract must be redistributed. This possibility means gas consumption.

Because of the EVM design, zero value control is the most error-prone value control, as zero value is assigned in case of no value input.

Also, since the constant value will be changed once, adding a zero-value control will not cause high gas consumption.

I recommend adding a zero check for literals when assigning critical values.

4 results - 4 files:

```solidity
contracts\Equity.sol:

  93:     constructor(Frankencoin zchf_) ERC20(18) {
  94:         zchf = zchf_;
  95:     }

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L93-L95


```solidity
contracts\Frankencoin.sol:

  59:    constructor(uint256 _minApplicationPeriod) ERC20(18){
  60:       MIN_APPLICATION_PERIOD = _minApplicationPeriod;
  62:    }

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L59-L62


```solidity
contracts\MintingHub.sol:

  54:     constructor(address _zchf, address factory) {
  55:         zchf = IFrankencoin(_zchf);
  56:         POSITION_FACTORY = IPositionFactory(factory);
  57:     }

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L54-L57


```solidity
contracts\Position.sol:

 50:     constructor(address _owner, address _hub, address _zchf, address _collateral, 
  54:         setOwner(_owner);  
  56:         hub = _hub;
  58:         zchf = IFrankencoin(_zchf);
  59:         collateral = IERC20(_collateral);
  60:         mintingFeePPM = _mintingFeePPM;
  61:         reserveContribution = _reservePPM;
  62:         minimumCollateral = _minCollateral;
  63:         challengePeriod = _challengePeriod;
  64:         start = block.timestamp + initPeriod; // one week time to deny the position
  66:         expiration = start + _duration;

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L50-L70


```solidity
contracts\StablecoinBridge.sol:

 26:     constructor(address other, address zchfAddress, uint256 limit_){
  27:         chf = IERC20(other);
  28:         zchf = IFrankencoin(zchfAddress);
  30:         limit = limit_;
  31:     }

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L26-L31

###  [G-06] Avoid contract existence checks by using solidity version 0.8.10 or later

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value.


There are 36 instances on the subject. (36 * 100 gas = 3.6k gas saved)

```solidity
contracts\Equity.sol:

  // @audit equity()
  109:    return VALUATION_FACTOR * zchf.equity() * ONE_DEC18 / totalSupply();

  // @audit equity()
  243:         uint256 equity = zchf.equity();

  // @audit equity()
  263:         return calculateSharesInternal(zchf.equity(), investment);

  // @audit transfer()
  279:         zchf.transfer(target, proceeds);

  // @audit equity()
  292:         uint256 capital = zchf.equity();

  // @audit equity()
  310:         require(zchf.equity() < MINIMUM_EQUITY);

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L109


```solidity
contracts\ERC20.sol:

  // @audit onTokenTransfer()
  165:    success = IERC677Receiver(recipient).onTokenTransfer(msg.sender, amount, data);

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L165


```solidity
contracts\MintingHub.sol:

  // @audit createNewPosition()
  93:             POSITION_FACTORY.createNewPosition(

  // @audit transferFrom()
  108:         zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE);

  // @audit isPosition()
  116:         require(zchf.isPosition(position) == address(this), "not our pos");

  // @audit reduceLimitForClone()
  126:         uint256 limit = existing.reduceLimitForClone(_initialMint);

  // @audit clonePosition()
  127:         address pos = POSITION_FACTORY.clonePosition(position);

  // @audit transferFrom()
  129:         existing.collateral().transferFrom(msg.sender, address(pos), _initialCollateral);

  // @audit price()  
  130:         IPosition(pos).initializeClone(msg.sender, existing.price(), limit, _initialCollateral, _initialMint);

  // @audit transferFrom()
  142:         IERC20(position.collateral()).transferFrom(msg.sender, address(this), _collateralAmount);

  // @audit challengePeriod()
  144:         challenges.push(Challenge(msg.sender, position, _collateralAmount, block.timestamp + position.challengePeriod(), address(0x0), 0));
  
  // @audit minimumCollateral()  
  170:         uint256 min = IPosition(challenge.position).minimumCollateral();  

  // @audit transfer()
  204:             zchf.transfer(challenge.bidder, challenge.bid); // return old bid

  // @audit transferFrom()
  210:             zchf.transferFrom(msg.sender, challenge.challenger, _bidAmountZCHF);

  // @audit transferFrom()
  225:             zchf.transferFrom(msg.sender, address(this), _bidAmountZCHF);

  // @audit transfer()
  263:             IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);

  // @audit transfer()
  272:         zchf.transfer(challenge.challenger, reward); // pay out the challenger reward

  // @audit transfer()
  284:         IERC20(collateral).transfer(target, amount);

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L93


```solidity
contracts\Position.sol:

  // @audit reserve()
  111:         IReserve(zchf.reserve()).checkQualified(msg.sender, helpers);

  // @audit transferFrom()
  138:             collateral.transferFrom(msg.sender, address(this), newCollateral - colbal);

  // @audit burnFrom()
  142:             zchf.burnFrom(msg.sender, minted - newMinted, reserveContribution);

  // @audit balanceOf()
  170:         return IERC20(collateral).balanceOf(address(this));

  // @audit calculateCurrentFee()
  195:         zchf.mint(target, amount, reserveContribution, calculateCurrentFee());

  // @audit transferFrom()
  228:         IERC20(zchf).transferFrom(msg.sender, address(this), amount);

  // @audit burnWithReserve()
  233:         uint256 actuallyBurned = IFrankencoin(zchf).burnWithReserve(burnable, reserveContribution);

  // @audit transfer()
  253:             IERC20(token).transfer(target, amount);

  // @audit transfer()
  269:         IERC20(collateral).transfer(target, amount);

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L111


```solidity
contracts\PositionFactory.sol:

  // @audit createClone()
  32:         Position clone = Position(createClone(existing.original()));

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L32


```solidity
contracts\StablecoinBridge.sol:

  // @audit transferFrom()
  45:         chf.transferFrom(msg.sender, address(this), amount);

  // @audit balanceOf()
  51:         require(chf.balanceOf(address(this)) <= limit, "limit");

  // @audit transfer()
  69:         chf.transfer(target, amount);

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L45


### [G-07] if () / require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

1 result - 1 file:
```solidity
contracts\Equity.sol:

  290:     function calculateProceeds(uint256 shares) public view returns (uint256) {
  291:         uint256 totalShares = totalSupply();
  292:         uint256 capital = zchf.equity();
  293:         require(shares + ONE_DEC18 < totalShares, "too many shares"); // make sure there is always at least one share
  294:         uint256 newTotalShares = totalShares - shares;
  295:         uint256 newCapital = _mulD18(capital, _power3(_divD18(newTotalShares, totalShares)));
  296:         return capital - newCapital;
  297:     }

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L290-L297


```diff
contracts\Equity.sol:

  290:     function calculateProceeds(uint256 shares) public view returns (uint256) {
  291:         uint256 totalShares = totalSupply();
+ 293:         require(shares + ONE_DEC18 < totalShares, "too many shares"); // make sure there is always at least one share
+ 294:         uint256 newTotalShares = totalShares - shares;
- 292:         uint256 capital = zchf.equity();
- 293:         require(shares + ONE_DEC18 < totalShares, "too many shares"); // make sure there is always at least one share
- 294:         uint256 newTotalShares = totalShares - shares;
+ 292:         uint256 capital = zchf.equity();
  295:         uint256 newCapital = _mulD18(capital, _power3(_divD18(newTotalShares, totalShares)));
  296:         return capital - newCapital;
  297:     }

```
### [G-08] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

3 results - 2 files:
```solidity
contracts\Frankencoin.sol:

   85:       if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
  
  267:       if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L85


```solidity
contracts\Position.sol:
  294:         if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();

```

**Recomendation Code:**
```diff
contracts\Position.sol:

- 294:         if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();
+              if (size < minimumCollateral) {
+                  if (size < collateralBalance()) {
+                      revert ChallengeTooSmall();
+                 }
+              }

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L294


### [G-09] Ternary operation is cheaper than if-else statement

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

2 results - 1 files:
```solidity
contracts\Frankencoin.sol:

  138:    function equity() public view returns (uint256) {
  139:       uint256 balance = balanceOf(address(reserve));
  140:       uint256 minReserve = minterReserve();
  141:       if (balance <= minReserve){
  142:         return 0;
  143:       } else {
  144:         return balance - minReserve;
  145:       }
  146:     }

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L138-L146


```diff
contracts\Frankencoin.sol:

  138:    function equity() public view returns (uint256) {
  139:       uint256 balance = balanceOf(address(reserve));
  140:       uint256 minReserve = minterReserve();
- 141:       if (balance <= minReserve){
- 142:         return 0;
- 143:       } else {
- 144:         return balance - minReserve;
- 145:       }
+            return (balance <= minReserve) ? 0 : (balance - minReserve);
  146:     }

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L120-L126

```solidity
contracts\Position.sol:

  120:     function getUsableMint(uint256 totalMint, bool afterFees) public view returns (uint256){
  121:         if (afterFees){
  122:             return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
  123:         } else {
  124:             return totalMint * (1000_000 - reserveContribution) / 1000_000;
  125:         }
  126:     }

```

```diff
contracts\Position.sol:

  120:     function getUsableMint(uint256 totalMint, bool afterFees) public view returns (uint256){
- 121:         if (afterFees){
- 122:             return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
- 123:         } else {
- 124:             return totalMint * (1000_000 - reserveContribution) / 1000_000;
- 125:         }
+              return afterFees ? totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000 : totalMint * (1000_000 - reserveContribution) / 1000_000;
  126:     }

### [G-10] Upgrade Solidity's optimizer

Make sure Solidity’s optimizer is enabled. It reduces gas costs. If you want to gas optimize for contract deployment (costs less to deploy a contract) then set the Solidity optimizer at a low number. If you want to optimize for run-time gas costs (when functions are called on a contract) then set the optimizer to a high number.


```js
hardhat.config.ts:
   93:   solidity: {
   94:       version: "0.8.13",
   95:       settings: {
   96:           optimizer: {
   97:               enabled: true,
   98:               runs: 200
   99:               },
  100:               outputSelection: {
  101:               "*": {
  102:                       "*": ["storageLayout"]
  103:                   }
  104:               }
  105:       }
  106:   },

```