# LOW FINDINGS

##

## [L-1] Loss of precision due to rounding

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

80:  price = _mint * ONE_DEC18 / _coll;
98:  uint256 reduction = (limit - minted - _minimum)/2; 
122: return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
124: return totalMint * (1000_000 - reserveContribution) / 1000_000;
187: return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));
239: return 1000000 * amountExcludingReserve / (1000000 - adjustedReservePPM); // 41 / (1-18%) = 50

```
[Position.sol#L80](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L80)

```solidity
FILE: FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

118: return minterReserveE6 / 1000000;

```
[Frankencoin.sol#L118](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L118)

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

109:  return VALUATION_FACTOR * zchf.equity() * ONE_DEC18 / totalSupply();
161:  voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); 
173:  return uint64(block.number << BLOCK_TIME_RESOLUTION_BITS);

```
[Equity.sol#L109](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L109)

```solidity 
FILE: 2023-04-frankencoin/contracts/MintingHub.sol

189:   return (challenge.bid * 1005) / 1000;

```
[MintingHub.sol#L189](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L189)

##

## [L-2] Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256

Using the SafeCast library can help prevent unexpected errors in your Solidity code and make your contracts more secure

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

187: return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));

```
[Position.sol#L187](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L187)

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

146:   totalVotesAtAnchor = uint192(totalVotes() - roundingLoss - lostVotes);
161:   voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); 

```
[Equity.sol#L146](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L146)

### Recommended Mitigation Steps:
Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256.

##

## [L-3] Vulnerable to cross-chain replay attacks due to static DOMAIN_SEPARATOR/domainSeparator

See this [issue](https://github.com/code-423n4/2021-04-maple-findings/issues/2) from a prior contest for details

```solidity
FILE: 2023-04-frankencoin/contracts/ERC20PermitLight.sol

function DOMAIN_SEPARATOR() public view returns (bytes32) {
        return
            keccak256(
                abi.encode(
                    //keccak256("EIP712Domain(uint256 chainId,address verifyingContract)");
                    bytes32(0x47e79534a245952e8b16893a336b85a3d9ea9fa8c573f3d803afb92a79469218),
                    block.chainid,
                    address(this)
                )
            );

```
[ERC20PermitLight.sol#L61-L70](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L61-L70)

## [L-4] MIXING AND OUTDATED COMPILER

The pragma version used are: 0.8.0

The minimum required version must be 0.8.17; otherwise, contracts will be affected by the following important bug fixes:

0.8.14:

ABI Encoder: When ABI-encoding values from calldata that contain nested arrays, correctly validate the nested array length against calldatasize() in all cases.
Override Checker: Allow changing data location for parameters only when overriding external functions.

0.8.15

Code Generation: Avoid writing dirty bytes to storage when copying bytes arrays.
Yul Optimizer: Keep all memory side-effects of inline assembly blocks.

0.8.16

Code Generation: Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.

0.8.17
Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.
Apart from these, there are several minor bug fixes and improvements

```solidity
FILE: 2023-04-frankencoin/contracts/ERC20.sol

12: pragma solidity ^0.8.0;

FILE: 2023-04-frankencoin/contracts/ERC20PermitLight.sol

5: pragma solidity ^0.8.0;

FILE: 2023-04-frankencoin/contracts/StablecoinBridge.sol

2: pragma solidity ^0.8.0;

FILE: 2023-04-frankencoin/contracts/StablecoinBridge.sol

2: pragma solidity ^0.8.0;

FILE: 2023-04-frankencoin/contracts/PositionFactory.sol

2: pragma solidity ^0.8.0;

FILE: 2023-04-frankencoin/contracts/MathUtil.sol

3: pragma solidity >=0.8.0 <0.9.0;

FILE: 2023-04-frankencoin/contracts/Ownable.sol

9: pragma solidity ^0.8.0;

FILE: 2023-04-frankencoin/contracts/Position.sol

2: pragma solidity ^0.8.0;

FILE: 2023-04-frankencoin/contracts/MintingHub.sol

2: pragma solidity ^0.8.0;

FILE: 2023-04-frankencoin/contracts/Equity.sol

4: pragma solidity >=0.8.0 <0.9.0;

FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

2: pragma solidity ^0.8.0;

```

##

## [L-5] abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()

Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). "Unless there is a compelling reason, abi.encode should be preferred". If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead. If all arguments are strings and or bytes, bytes.concat() should be used instead

```solidity
FILE: 2023-04-frankencoin/contracts/ERC20PermitLight.sol

abi.encodePacked(
                        "\x19\x01",
                        DOMAIN_SEPARATOR(),
                        keccak256(
                            abi.encode(
                                // keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),
                                bytes32(0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9),
                                owner,
                                spender,
                                value,
                                nonces[owner]++,
                                deadline
                            )
                        )
                    )
                ),
                v,
                r,
                s
            );

```
[ERC20PermitLight.sol#L35-L54](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L35-L54)

##

## [L-6] Lack of Sanity/Threshold/Limit Checks

Devoid of sanity/threshold/limit checks, critical parameters can be configured to invalid values, causing a variety of issues and breaking expected interactions within/between contracts. Consider adding proper uint256 validation as well as zero address checks for critical changes. A worst case scenario would render the contract needing to be re-deployed in the event of human/accidental errors that involve value assignments to immutable variables. If the validation procedure is unclear or too complex to implement on-chain, document the potential issues that could produce invalid values

```solidity
FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

_minApplicationPeriod value is not checked before assigning to MIN_APPLICATION_PERIOD 

constructor(uint256 _minApplicationPeriod) ERC20(18){
      MIN_APPLICATION_PERIOD = _minApplicationPeriod;
      reserve = new Equity(this);
   }

function registerPosition(address _position) override external {
      if (!isMinter(msg.sender)) revert NotMinter();
      positions[_position] = msg.sender;
   }

function burn(uint256 _amount) external {
      _burn(msg.sender, _amount);
   }

function isMinter(address _minter) override public view returns (bool){
      return minters[_minter] != 0 && block.timestamp >= minters[_minter];
   }

```
[Frankencoin.sol#L59-L62](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L59-L62)

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

112: function _beforeTokenTransfer(address from, address to, uint256 amount) override internal {
113: super._beforeTokenTransfer(from, to, amount);

```
[Equity.sol#L112-L113](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L112-L113)

```solidity 
FILE: 2023-04-frankencoin/contracts/MintingHub.sol

function openPosition(
        address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
        uint256 _mintingMaximum, uint256 _initPeriodSeconds, uint256 _expirationSeconds, uint256 _challengeSeconds,
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address) {

```
[MintingHub.sol#L88-L105](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L88-L105)

##

## [L-7] 

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

```solidity
FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

31: IReserve override public immutable reserve;

```
[Frankencoin.sol#L31](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L31)

##

## [NC-2] For modern and more readable code; update import usages

### Context
All In Scope Contracts 

### Description
Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.

This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

### Recommendation
import {contract1 , contract2} from "filename.sol";

##

## [NC-3] Missing NATSPEC

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L181-L189

##

## [NC-4] For functions, follow Solidity standard naming conventions (internal function style rule)

### Description
The above codes don’t follow Solidity’s standard naming convention,

internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

193: function mintInternal(address target, uint256 amount, uint256 collateral_) internal {
202: function restrictMinting(uint256 period) internal {
232: function repayInternal(uint256 burnable) internal {
240: function notifyRepaidInternal(uint256 amount) internal 
268: function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {
282: function checkCollateral(uint256 collateralReserve, uint256 atPrice) internal view {
286: function emitUpdate() internal {

```
[Position.sol#L193](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L193)

```solidity
FILE: FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

102: function allowanceInternal(address owner, address spender) internal view override returns (uint256) {

```
[Frankencoin.sol#L102](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L102)

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

144:  function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {
157:  function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){

```
[Equity.sol#L144](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L144)

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

## [NC-5] Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)

While the compiler knows to optimize away the exponentiation, it’s still better coding practice to use idioms that do not require compiler optimization, if they exist

```solidity
FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

25: uint256 public constant MIN_FEE = 1000 * (10**18);

```
[Frankencoin.sol#L25](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L25)

```solidity 
FILE: 2023-04-frankencoin/contracts/MintingHub.sol

20:     uint256 public constant OPENING_FEE = 1000 * 10**18;

```
[MintingHub.sol#L20](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L20)

##

## [NC-6] 




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