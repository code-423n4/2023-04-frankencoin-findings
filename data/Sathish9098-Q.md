# LOW FINDINGS

# NON CRITICAL FINDINGS

##

## [NC-1] immutable should be uppercase 

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol
uint256 public immutable start; // timestamp when minting can start
uint256 public immutable expiration; // timestamp at which the position expires
address public immutable original; // originals point to themselves, clone to their origin
address public immutable hub; // the hub this position was created by
IFrankencoin public immutable zchf; // currency
IERC20 public override immutable collateral; // collateral
uint256 public override immutable minimumCollateral; // prevent dust amounts
uint32 public immutable mintingFeePPM;
uint32 public immutable reserveContribution; // in ppm

```
[Position.sol#L29-L39](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L29-L39)

##

## [NC-2] For modern and more readable code; update import usages

### Context
All In Scope Contracts 

### Description
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.

This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

### Recommendation
import {contract1 , contract2} from "filename.sol";






LOW‑1	Use of ecrecover is susceptible to signature malleability	1
LOW‑2	Event is missing parameters	2
LOW‑3	Possible rounding issue	1
LOW‑4	Low Level Calls With Solidity Version 0.8.14 Can Result In Optimiser Bug	1
LOW‑5	Minting tokens to the zero address should be avoided	1
LOW‑6	Missing Checks for Address(0x0)	1
LOW‑7	Prevent division by 0	2
LOW‑8	Use safetransfer Instead Of transfer	20
LOW‑9	Admin privilege - A single point of failure can allow a hacked or malicious owner use critical functions in the project	7
LOW‑10	TransferOwnership Should Be Two Step	1
LOW‑11	Unbounded loop	2
LOW‑12	Use safeTransferOwnership instead of transferOwnership function	1

NC‑1	Add a timelock to critical functions	1
NC‑2	Avoid Floating Pragmas: The Version Should Be Locked	8
NC‑3	Constants Should Be Defined Rather Than Using Magic Numbers	5
NC‑4	Critical Changes Should Use Two-step Procedure	1
NC‑5	Declare interfaces on separate files	1
NC‑6	Duplicated require()/revert() Checks Should Be Refactored To A Modifier Or Function	4
NC‑7	Event Is Missing Indexed Fields	3
NC‑8	Function writing that does not comply with the Solidity Style Guide	10
NC‑9	Large or complicated code bases should implement fuzzing tests	1
NC‑10	Imports can be grouped together	11
NC‑11	NatSpec return parameters should be included in contracts	1
NC‑12	Initial value check is missing in Set Functions	1
NC‑13	Lines are too long	1
NC‑14	Implementation contract may not be initialized	6
NC‑15	NatSpec comments should be increased in contracts	1
NC‑16	Non-usage of specific imports	28
NC‑17	Use a more recent version of Solidity	10
NC‑18	Open TODOs	1
NC‑19	Using >/>= without specifying an upper bound is unsafe	2
NC‑20	Public Functions Not Called By The Contract Should Be Declared External Instead	3
NC‑21	Empty blocks should be removed or emit something	1
NC‑22	require() / revert() Statements Should Have Descriptive Reason Strings	10
NC‑23	Large multiples of ten should use scientific notation	6
NC‑24	Use bytes.concat()	1
NC‑25	Use Underscores for Number Literals	6