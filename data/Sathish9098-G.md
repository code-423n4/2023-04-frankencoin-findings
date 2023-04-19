# GAS OPTIMIZATION

##

## [G-1] State variables should be cached in stack variables rather than re-reading them from storage

Caching will replace each Gwarmaccess (100 gas) with a much cheaper stack read.
Less obvious fixes/optimizations include having local storage variables of mappings within state variable mappings or mappings within state variable structs, having local storage variables of structs within mappings, having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.


```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

minted state variable should be cached with stack variable 

141: if (newMinted < minted){ /// @audit minted cached
            zchf.burnFrom(msg.sender, minted - newMinted, reserveContribution); /// @audit minted cached
            minted = newMinted;
        }
        if (newCollateral < colbal){
            withdrawCollateral(msg.sender, colbal - newCollateral);
        }
        // Must be called after collateral withdrawal
        if (newMinted > minted){  /// @audit minted cached
            mint(msg.sender, newMinted - minted);  /// @audit minted cached
        }

241: if (amount > minted) revert RepaidTooMuch(amount - minted);
        minted -= amount;

349:  uint256 repayment = minted < volumeZCHF ? minted : volumeZCHF; 

```
[Position.sol#L141-L150](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L141-L150)

##

## [G-2] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
FILE: 2023-04-frankencoin/contracts/MintingHub.sol

159: Challenge memory copy = Challenge(

```
[MintingHub.sol#L159](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L159)


## [G-3] For events use 3 indexed rule to save gas 

> Instances()

Need to declare 3 indexed fields for event parameters. If the event parameter is less than 3 should declare all event parameters indexed 

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

41: event PositionOpened(address indexed owner, address original, address zchf, address collateral, uint256 price);
42: event MintingUpdate(uint256 collateral, uint256 price, uint256 minted, uint256 limit);
43: event PositionDenied(address indexed sender, string message); // emitted if closed by governance

```
[Position.sol#L41-L43](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L41-L43)

```solidity 
FILE: 2023-04-frankencoin/contracts/MintingHub.sol

48: event ChallengeStarted(address indexed challenger, address indexed position, uint256 size, uint256 number);
49: event ChallengeAverted(address indexed position, uint256 number);
50: event ChallengeSucceeded(address indexed position, uint256 bid, uint256 number);
51: event NewBid(uint256 challengedId, uint256 bidAmount, address bidder);
52: event PostPonedReturn(address collateral, address indexed beneficiary, uint256 amount);

```
[MintingHub.sol#L48-L52](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L48-L52)

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

90: event Delegation(address indexed from, address indexed to); // indicates a delegation
91: event Trade(address who, int amount, uint totPrice, uint newprice); // amount pos or neg for mint or redemption

```
[Equity.sol#L90-L91](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L90-L91)

```solidity
FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

52: event MinterApplied(address indexed minter, uint256 applicationPeriod, uint256 applicationFee, string message);
53: event MinterDenied(address indexed minter, string message);

```
[Frankencoin.sol#L52-L53](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L52-L53)

##

## [G-4] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to [not having to recalculate the key’s keccak256 hash](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0) (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

```solidity
FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

45: mapping (address => uint256) public minters;
50: mapping (address => address) public positions;

```
[Frankencoin.sol#L45-L50](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L45-L50)

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

83: mapping (address => address) public delegates;
88: mapping (address => uint64) private voteAnchor;

```
[Equity.sol#L83-L88](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L83-L88)

```solidity
FILE: 2023-04-frankencoin/contracts/ERC20.sol

43: mapping (address => uint256) private _balances;
45: mapping (address => mapping (address => uint256)) private _allowances;

```
[ERC20.sol#L43-L45](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L43-L45)

##

## [G-5] Lack of input value checks cause a redeployment if any human/accidental errors

Devoid of sanity/threshold/limit checks, critical parameters can be configured to invalid values, causing a variety of issues and breaking expected interactions within/between contracts. Consider adding proper uint256 validation. A worst case scenario would render the contract needing to be re-deployed in the event of human/accidental errors that involve value assignments to immutable variables.

If any human/accidental errors happen need to redeploy the contract so this create the huge gas lose 

```solidity
FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

_minApplicationPeriod value is not checked before assigning to MIN_APPLICATION_PERIOD 

constructor(uint256 _minApplicationPeriod) ERC20(18){
      MIN_APPLICATION_PERIOD = _minApplicationPeriod;
      reserve = new Equity(this);
   }

```
[Frankencoin.sol#L59-L62](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L59-L62)

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

constructor(address _owner, address _hub, address _zchf, address _collateral, 
        uint256 _minCollateral, uint256 _initialLimit, uint256 initPeriod, uint256 _duration,
        uint256 _challengePeriod, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) {
        require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values
        setOwner(_owner);
        original = address(this);
        hub = _hub;
        price = _liqPrice;
        zchf = IFrankencoin(_zchf);
        collateral = IERC20(_collateral);
        mintingFeePPM = _mintingFeePPM;
        reserveContribution = _reservePPM;
        minimumCollateral = _minCollateral;
        challengePeriod = _challengePeriod;
        start = block.timestamp + initPeriod; // one week time to deny the position
        cooldown = start;
        expiration = start + _duration;
        limit = _initialLimit;
        
        emit PositionOpened(_owner, original, _zchf, address(collateral), _liqPrice);
    }

```
[Position.sol#L50-L70](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L50-L70)

```solidity
FILE: 2023-04-frankencoin/contracts/MintingHub.sol

constructor(address _zchf, address factory) {
        zchf = IFrankencoin(_zchf);
        POSITION_FACTORY = IPositionFactory(factory);
    }

```
[MintingHub.sol#L54-L57](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L54-L57)

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

 constructor(Frankencoin zchf_) ERC20(18) {
        zchf = zchf_;
    }

```
[Equity.sol#L93-L95](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L93-L95)

```solidity
FILE: 2023-04-frankencoin/contracts/StablecoinBridge.sol

constructor(address other, address zchfAddress, uint256 limit_){
        chf = IERC20(other);
        zchf = IFrankencoin(zchfAddress);
        horizon = block.timestamp + 52 weeks;
        limit = limit_;
    }

```
[StablecoinBridge.sol#L26-L31](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L26-L31)

##

## [G-6] Use nested if and, avoid multiple check combinations

> Instances()

> Approximate Gas Saved ( )

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

As per Solidity [reports](https://gist.github.com/sathishpic22/fe96671bafb22ceaace7fc05a66bd115) possible to save 9 gas 

```solidity
FILE: FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

84:   if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
85:   if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
267:  if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();

```
[Frankencoin.sol#L84-L85](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L84-L85)

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

294:  if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();

```
[Position.sol#L294](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L294)

##

## [G-7] No need to evaluate all expressions to know if one of them is true

When we have a code expressionA || expressionB if expressionA is true then expressionB will not be evaluated and gas saved

```solidity
FILE: FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

106: } else if (isMinter(spender) || isMinter(isPosition(spender))){
```
[Frankencoin.sol#L106](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L106)

##

## [G-8] Repeated functions should be cached instead of multiple calls to save gas 

In Solidity, caching repeated function calls can be an effective way to optimize gas usage, especially when the function is called frequently with the same arguments

```solidity
FILE: FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

totalSupply() value should be cached instead of calling multiple times 

84: if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
85: if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
```
[Frankencoin.sol#L84-L85](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L84-L85)


```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

anchorTime() function results should be cached 

function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {
        uint256 lostVotes = from == address(0x0) ? 0 : (anchorTime() - voteAnchor[from]) * amount;
        totalVotesAtAnchor = uint192(totalVotes() - roundingLoss - lostVotes);
        totalVotesAnchorTime = anchorTime();
    }
```
[Equity.sol#L144-L148](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L144-L148)

##

## [G-9] Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it’s not consistently done in the solution.
I suggest adding a non-zero-value check here

```solidity
FILE: FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

87:  _transfer(msg.sender, address(reserve), _applicationFee);
283: _transfer(address(reserve), msg.sender, _amount);
285: _transfer(address(reserve), msg.sender, reserveLeft);
254: _transfer(address(reserve), msg.sender, freedAmount - _amountExcludingReserve); 
```
[Frankencoin.sol#L87](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L87)

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

279:    zchf.transfer(target, proceeds);
```
[Equity.sol#L279](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L279)

```solidity 
FILE: 2023-04-frankencoin/contracts/MintingHub.sol

108:  zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE);
129:  existing.collateral().transferFrom(msg.sender, address(pos), _initialCollateral);
225:  zchf.transferFrom(msg.sender, address(this), _bidAmountZCHF);

```
[MintingHub.sol#L108](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L108)
##

## [G-10] Don't declare the variable inside the loops 

In every iterations the new variables instance created this will consumes more gas . So just declare variables outside the loop and only use inside to save gas 

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

for (uint i=0; i<helpers.length; i++){
            address current = helpers[i];

for (uint256 i = 0; i<addressesToWipe.length; i++){
            address current = addressesToWipe[0];

``` 
[Equity.sol#L192-L193](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L192-L193)

##

## [G-11] Empty blocks should be removed to save deployment cost 

```solidity

FILE: ERC20.sol

240: function _beforeTokenTransfer(address from, address to, uint256 amount) virtual internal {
    }

```
[ERC20.sol#L240](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/ERC20.sol#L240)

##

## [G-12] Functions should be used instead of modifiers to save gas 

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

modifier noCooldown() {
        if (block.timestamp <= cooldown) revert Hot();
        _;
    }


    modifier noChallenge() {
        if (challengedAmount > 0) revert Challenged();
        _;
    }


    modifier onlyHub() {
        if (msg.sender != address(hub)) revert NotHub();
        _;
    }

```
[Position.sol#L373-L390](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L373-L390)

##

## [G-13] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

109:  return VALUATION_FACTOR * zchf.equity() * ONE_DEC18 / totalSupply();
243:  uint256 equity = zchf.equity();
279:  zchf.transfer(target, proceeds);
292:  uint256 capital = zchf.equity();
310:  require(zchf.equity() < MINIMUM_EQUITY); 

```
[Equity.sol#L109](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L109)

##

## [G-14] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```

//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)

```

```solidity
FILE: 2023-04-frankencoin/contracts/MintingHub.sol



```







GAS‑1	abi.encode() is less efficient than abi.encodepacked()	2	200
GAS‑2	<array>.length Should Not Be Looked Up In Every Loop Of A For-loop	3	291
GAS‑3	Setting the constructor to payable	6	78
GAS‑4	Do not calculate constants	7	-
GAS‑5	Duplicated require()/revert() Checks Should Be Refactored To A Modifier Or Function	4	112
GAS‑6	++i Costs Less Gas Than i++, Especially When It’s Used In For-loops (--i/i-- Too)	2	12
GAS‑7	Use assembly to write address storage values	1	-
GAS‑8	Functions guaranteed to revert when called by normal users can be marked payable	6	126
GAS‑9	Use hardcoded address instead address(this)	9	-
GAS‑10	It Costs More Gas To Initialize Variables To Zero Than To Let The Default Of Zero Be Applied	1	-
GAS‑11	internal functions only called once can be inlined to save gas	14	308
GAS‑12	Multiplication/division By Two Should Use Bit Shifting	1	-
GAS‑13	Optimize names to save gas	10	220
GAS‑14	<x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables	20	-
GAS‑15	Structs can be packed into fewer storage slots by editing time variables	2	4000
GAS‑16	Using private rather than public for constants, saves gas	5	-
GAS‑17	Public Functions To External	37	-
GAS‑18	Save gas with the use of specific import statements	28	-
GAS‑19	Splitting require() Statements That Use && Saves Gas	1	9
GAS‑20	State variables can be packed into fewer storage slots	1	2000
GAS‑21	Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead	10	-
GAS‑22	++i/i++ Should Be unchecked{++i}/unchecked{i++} When It Is Not Possible For Them To Overflow, As Is The Case When Used In For- And While-loops	3	105
GAS‑23	Using unchecked blocks to save gas	2	40
GAS‑24	Unnecessary look up in if condition	1	2100
GAS‑25	Use of Custom Errors Instead of String	12	-
GAS‑26	Use solidity version 0.8.19 to gain some gas boost	10	880
GAS‑27	Using 10**X for constants isn't gas efficient	3	-