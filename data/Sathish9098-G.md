# GAS OPTIMIZATION

## [G-1] For events use 3 indexed rule to save gas 

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

## [G-2] Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate

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

##

## [G-3] Lack of input value checks cause a redeployment if any human/accidental errors

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

##

## [G-4] Use nested if and, avoid multiple check combinations

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
```
##

## [G-5] No need to evaluate all expressions to know if one of them is true

When we have a code expressionA || expressionB if expressionA is true then expressionB will not be evaluated and gas saved

```solidity
FILE: FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

106: } else if (isMinter(spender) || isMinter(isPosition(spender))){
```
[Frankencoin.sol#L106](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L106)

```solidity
```
```solidity
```
```solidity
```
```solidity
```
```solidity
```
```solidity
```
##

## [G-6] Repeated functions should be cached instead of multiple calls to save gas 

In Solidity, caching repeated function calls can be an effective way to optimize gas usage, especially when the function is called frequently with the same arguments

```solidity
FILE: FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

totalSupply() value should be cached instead of calling multiple times 

84: if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
85: if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
```
[Frankencoin.sol#L84-L85](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L84-L85)

```solidity
```
```solidity
```
```solidity
```
```solidity
```
```solidity
```
```solidity
```

##

## [G-7] Before transfers funds the amount must be checked with zero values to save gas 

Checking whether the amount being transferred is zero before executing a transfer can be a useful optimization technique to save gas in Solidity. This is because executing a transfer with a zero value will still consume gas, but it will not actually transfer any funds

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

## [G-8] Don't declare the variable inside the loops 

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

## [G-9] Empty blocks should be removed to save deployment cost 

```solidity

FILE: ERC20.sol

240: function _beforeTokenTransfer(address from, address to, uint256 amount) virtual internal {
    }

```
[ERC20.sol#L240](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/ERC20.sol#L240)

##

## [G-10] 


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