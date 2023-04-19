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

## [L-7] Function Calls in Loop Could Lead to Denial of Service

Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to the gas limitations or failed transactions 


```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

for (uint i=0; i<helpers.length; i++){
            address current = helpers[i];

 for (uint256 i = 0; i<addressesToWipe.length; i++){
            address current = addressesToWipe[0];
```
[Equity.sol#L192](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L192)

##

## [L-8] Use .call instead of .transfer to send ether

.transfer will relay 2300 gas and .call will relay all the gas. If the receive/fallback function from the recipient proxy contract has complex logic, using .transfer will fail, causing integration issues

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

279: zchf.transfer(target, proceeds);

```
[Equity.sol#L279](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L279)

```solidity
FILE: 2023-04-frankencoin/contracts/MintingHub.sol

294:  challenge.position.collateral().transfer(challenge.challenger, challenge.size);
204:  zchf.transfer(challenge.bidder, challenge.bid); // return old bid
211:  challenge.position.collateral().transfer(msg.sender, challenge.size);
268:  zchf.transfer(owner, effectiveBid - fundsNeeded);

```
[MintingHub.sol#L294](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L294)

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

253: IERC20(token).transfer(target, amount);
269: IERC20(collateral).transfer(target, amount);

```
[Position.sol#L253](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L253)

##

## [L-9] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an ” EMERGENCY STOP (CIRCUIT BREAKER) PATTERN “.

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

##

## [L-10]  ALLOWS MALLEABLE SECP256K1 SIGNATURES 

Here, the ecrecover() method doesn’t check the s range.

Homestead (EIP-2) added this limitation, however the precompile remained unaltered. The majority of libraries, including OpenZeppelin, do this check.

Since an order can only be confirmed once and its hash is saved, there doesn’t seem to be a serious danger in existing use cases

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/7201e6707f6631d9499a569f492870ebdd4133cf/contracts/utils/cryptography/ECDSA.sol#L138-L149

```solidity
FILE: 2023-04-frankencoin/contracts/ERC20PermitLight.sol

 address recoveredAddress = ecrecover(
                keccak256(
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
[ERC20PermitLight.sol#L33-L54](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L33-L54)

##

## [L-11] AVOID HARDCODED VALUES

It is not good practice to hardcode values, but if you are dealing with addresses much less, these can change between implementations, networks or projects, so it is convenient to remove these values from the source code

```solidity
FILE: 2023-04-frankencoin/contracts/ERC20PermitLight.sol

41:  bytes32(0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9),
66:  bytes32(0x47e79534a245952e8b16893a336b85a3d9ea9fa8c573f3d803afb92a79469218),

```
[ERC20PermitLight.sol#L41](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L41)

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L41

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L43

##

## [L-12] Front running attacks by the onlyOwner

owner value is not a constant value and can be changed with transferOwnership() function, before a function using setOwner state variable value in the project, transferOwnership function can be triggered by onlyOwner and operations can be blocked

```solidity
FILE: 2023-04-frankencoin/contracts/Ownable.sol

 function transferOwnership(address newOwner) public onlyOwner {
        setOwner(newOwner);
    }

    /**
     * @dev Transfers ownership of the contract to a new account (`newOwner`).
     * Internal function without access restriction.
     */
    function setOwner(address newOwner) internal {
        address oldOwner = owner;
        owner = newOwner;
        emit OwnershipTransferred(oldOwner, newOwner);
    }


```

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

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L30

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L31

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L51



##

## [NC-2] Missing NATSPEC

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L181-L189

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L181-L189

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L292-L296

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L366-L390

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L54-L57

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L59-L65

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L235-L237

##

## [NC-3] For functions, follow Solidity standard naming conventions (internal function style rule)

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

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L39

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L45

##

## [NC-4] Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)

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

## [NC-5] Need Fuzzing test for unchecked 

```solidity
FILE: 2023-04-frankencoin/contracts/ERC20PermitLight.sol

32: unchecked { 

```
[ERC20PermitLight.sol#L32](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L32)

##

## [NC-6] Remove commented out code

```solidity
FILE: 2023-04-frankencoin/contracts/ERC20PermitLight.sol

65: //keccak256("EIP712Domain(uint256 chainId,address verifyingContract)");

```
[ERC20PermitLight.sol#L65](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L65)

##

## [NC-7] Inconsistent spacing in comments

Some lines use // x and some use //x. The instances below point out the usages that don’t follow the majority, within each file

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L65

##

## [NC-8] Inconsistent method of specifying a floating pragma

Some files use >=, some use ^. The instances below are examples of the method that has the fewest instances for a specific version. Note that using >= without also specifying <= will lead to failures to compile, or external project incompatability, when the major version changes and there are breaking-changes, so ^ should be preferred regardless of the instance counts

```solidity

FILE: 2023-04-frankencoin/contracts/PositionFactory.sol

2: pragma solidity ^0.8.0;

FILE: 2023-04-frankencoin/contracts/MathUtil.sol

3: pragma solidity >=0.8.0 <0.9.0;

```
##

## [NC-9] Numeric values having to do with time should use time units for readability

There are [units](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#time-units) for seconds, minutes, hours, days, and weeks, and since they’re defined, they should be used

```solidity
FILE: 2023-04-frankencoin/contracts/Equity.sol

51:  uint8 private constant BLOCK_TIME_RESOLUTION_BITS = 24;

59:  uint256 public constant MIN_HOLDING_DURATION = 90*7200 << BLOCK_TIME_RESOLUTION_BITS;

```
[Equity.sol#L59](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L59)

##

## [NC-10] NO SAME VALUE INPUT CONTROL 

```solidity
FILE : 2023-04-frankencoin/contracts/Ownable.sol

  function setOwner(address newOwner) internal {
        address oldOwner = owner;
        owner = newOwner;
        emit OwnershipTransferred(oldOwner, newOwner);
    }

```
[Ownable.sol#L39-L43](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L39-L43)

##

## [NC-11] Contract layout and order of functions

The Solidity style guide [recommends](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout)

Declare internal functions bellow the external/public functions 

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L268-L304

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L188-L199

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L266-L275

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L102-L125

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L97-L108

##

## [NC-12] Constant redefined elsewhere

Consider defining in only one contract so that values cannot become out of sync when only one location is updated.

A cheap way to store constants in a single location is to create an internal constant in a library. If the variable is a local cache of another contract’s value, consider making the cache variable internal or private, which will require external users to query the contract with the source of truth, so that callers don’t get out of sync.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L20-L26

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L39-L59

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MathUtil.sol#L10-L11

##

## [NC-13] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword

```solidity
FILE: 2023-04-frankencoin/contracts/ERC20PermitLight.sol

- 15: mapping(address => uint256) public nonces;
+ 15: mapping (address => uint256) public nonces;
```
[ERC20PermitLight.sol#L15](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L15)

##

## [NC-14] Tokens accidentally sent to the contract cannot be recovered

It can’t be recovered if the tokens accidentally arrive at the contract address, which has happened to many popular projects, so I recommend adding a recovery code to your critical contracts

### Recommended Mitigation Steps
Add this code:
```solidity
 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}
```

##

## [NC-15] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

##

## [NC-16] Constants on the left are better

If you use the constant first you support structures that veil programming errors. And one should only produce code either to add functionality, fix an programming error or trying to support structures to avoid programming errors (like design patterns).

https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L86

##

## [NC-17] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone

```solidity
FILE: 2023-04-frankencoin/contracts/PositionFactory.sol

39:   assembly {

```
[PositionFactory.sol#L39](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L39)

##

## [NC-18] Take advantage of Custom Error’s return value property

An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the () sign, this kind of approach provides a serious advantage in debugging and examining the revert details of dapps such as tenderly

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

77:  if(_coll < minimumCollateral) revert InsufficientCollateral();
110: if (block.timestamp >= start) revert TooLate();
194: if (minted + amount > limit) revert LimitExceeded();
294: if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();

```
[Position.sol#L77](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L77)

##

## [NC-19] Use constants instead of using numbers directly without explanations 

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

122: return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
124: return totalMint * (1000_000 - reserveContribution) / 1000_000;

```
[Position.sol#L122](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L122)

```solidity
FILE: 2023-04-frankencoin/contracts/MintingHub.sol

189: return (challenge.bid * 1005) / 1000;
217: uint256 earliestEnd = block.timestamp + 30 minutes;
265: uint256 reward = (volume * CHALLENGER_REWARD) / 1000_000;

```
[MintingHub.sol#L189](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L189)

```solidity
FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

118: return minterReserveE6 / 1000000;
166: uint256 usableMint = (_amount * (1000_000 - _feesPPM - _reservePPM)) / 1000_000; // rounding down is fine
205: uint256 theoreticalReserve = _reservePPM * mintedAmount / 1000000;
239: return 1000000 * amountExcludingReserve / (1000000 - adjustedReservePPM); // 41 / (1-18%) = 50


```
[Frankencoin.sol#L118](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L118)

##

## [NC-20] Shorthand way to write if / else statement

The normal if / else statement can be refactored in a shorthand way to write it:

Increases readability
Shortens the overall SLOC

```solidity
FILE: 2023-04-frankencoin/contracts/Position.sol

184: if (time >= exp){
            return 0;
        } else {
            return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));
        }

160: if (newPrice > price) {
            restrictMinting(3 days);
        } else {
            checkCollateral(collateralBalance(), newPrice);
        }

121: if (afterFees){
            return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
        } else {
            return totalMint * (1000_000 - reserveContribution) / 1000_000;
        }

250: if (token == address(collateral)){
            withdrawCollateral(target, amount);
        } else {
            IERC20(token).transfer(target, amount);
        }

```
[Position.sol#L184-L188](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L184-L188)

```solidity
FILE: 2023-04-frankencoin/contracts/MintingHub.sol

267: if (effectiveBid > fundsNeeded){
            zchf.transfer(owner, effectiveBid - fundsNeeded);
        } else if (effectiveBid < fundsNeeded){
            zchf.notifyLoss(fundsNeeded - effectiveBid); // ensure we have enough to pay everything
        }
```
[MintingHub.sol#L267-L271](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L267-L271)

```solidity
FILE: 2023-04-frankencoin/contracts/Frankencoin.sol

141: if (balance <= minReserve){
        return 0;
      } else {
        return balance - minReserve;
      }

207: if (currentReserve < minterReserve()){
         // not enough reserves, owner has to take a loss
         return theoreticalReserve * currentReserve / minterReserve();
      } else {
         return theoreticalReserve;
      }


```
[Frankencoin.sol#L141-L145](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L141-L145)

### Recommended Mitigation 

```solidity

time >= exp ? return 0 : return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));

```

##

## [NC-21] Return values of approve() not checked

Not all IERC20 implementations revert() when there’s a failure in approve(). The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

```solidity
FILE: 2023-04-frankencoin/contracts/ERC20.sol

109: _approve(msg.sender, spender, value);
132: _approve(sender, msg.sender, currentAllowance - amount);

```
[ERC20.sol#L109](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L109)




