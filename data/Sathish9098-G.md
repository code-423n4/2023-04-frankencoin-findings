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