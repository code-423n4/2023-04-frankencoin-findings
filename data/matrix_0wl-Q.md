## Non Critical Issues

|       | Issue                                                                   |
| ----- | :---------------------------------------------------------------------- |
| NC-1  | ADD TO INDEXED PARAMETER FOR COUNTABLE EVENTS                           |
| NC-2  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                                    |
| NC-3  | Be explicit declaring types                                             |
| NC-4  | GENERATE PERFECT CODE HEADERS EVERY TIME                                |
| NC-5  | USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS                         |
| NC-6  | SOLIDITY COMPILER VERSIONS MISMATCH                                     |
| NC-7  | SIGNATURE MALLEABILITY OF EVM’S `ECRECOVER()`                           |
| NC-8  | FUNCTIONS, PARAMETERS AND VARIABLES IN SNAKE CASE                       |
| NC-9  | FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES                 |
| NC-10 | LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION                   |
| NC-11 | FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION           |
| NC-12 | ALLOWS MALLEABLE SECP256K1 SIGNATURES                                   |
| NC-13 | MISSING EVENT FOR CRITICAL PARAMETER CHANGE                             |
| NC-14 | MISSING FEE PARAMETER VALIDATION                                        |
| NC-15 | NO SAME VALUE INPUT CONTROL                                             |
| NC-16 | FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE     |
| NC-17 | USE A MORE RECENT VERSION OF SOLIDITY                                   |
| NC-18 | Unused imports                                                          |
| NC-19 | FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS |

### [NC-1] ADD TO INDEXED PARAMETER FOR COUNTABLE EVENTS

#### Description:

Add to indexed parameter for countable Events

#### **Proof Of Concept**

```solidity
File: contracts/ERC20.sol

158:         emit Transfer(sender, recipient, amount);

186:         emit Transfer(address(0), recipient, amount);

205:         emit Transfer(account, address(0), amount);

223:         emit Approval(owner, spender, value);

```

### [NC-2] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: contracts/Ownable.sol

32:         setOwner(newOwner);

39:     function setOwner(address newOwner) internal {

```

```solidity
File: contracts/Position.sol

54:         setOwner(_owner);

78:         setOwner(owner);

```

### [NC-3] Be explicit declaring types

#### Description:

Instead of uint use uint256

#### **Proof Of Concept**

```solidity
File: contracts/Equity.sol

91:     event Trade(address who, int amount, uint totPrice, uint newprice); // amount pos or neg for mint or redemption

192:         for (uint i=0; i<helpers.length; i++){

196:             for (uint j=i+1; j<helpers.length; j++){

```

### [NC-4] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-5] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS

#### Description:

There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes.

Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

#### **Proof Of Concept**

```solidity
File: contracts/Equity.sol

39:     uint32 public constant VALUATION_FACTOR = 3;

59:     uint256 public constant MIN_HOLDING_DURATION = 90*7200 << BLOCK_TIME_RESOLUTION_BITS; // Set to 5 for local testing

```

```solidity
File: contracts/Frankencoin.sol

25:    uint256 public constant MIN_FEE = 1000 * (10**18);

```

```solidity
File: contracts/MintingHub.sol

20:     uint256 public constant OPENING_FEE = 1000 * 10**18;

26:     uint32 public constant CHALLENGER_REWARD = 20000; // 2%

```

### [NC-6] SOLIDITY COMPILER VERSIONS MISMATCH

#### Description:

The project is compiled with different versions of Solidity, which is not recommended because it can lead to undefined behaviors.

It is better to use one Solidity compiler version across all contracts instead of different versions with different bugs and security checks.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20.sol

12: pragma solidity ^0.8.0;

```

```solidity
File: contracts/ERC20PermitLight.sol

5: pragma solidity ^0.8.0;

```

```solidity
File: contracts/Equity.sol

4: pragma solidity >=0.8.0 <0.9.0;

```

```solidity
File: contracts/Frankencoin.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: contracts/MathUtil.sol

3: pragma solidity >=0.8.0 <0.9.0;

```

```solidity
File: contracts/MintingHub.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: contracts/Ownable.sol

9: pragma solidity ^0.8.0;

```

```solidity
File: contracts/Position.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: contracts/PositionFactory.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: contracts/StablecoinBridge.sol

2: pragma solidity ^0.8.0;

```

### [NC-7] SIGNATURE MALLEABILITY OF EVM’S `ECRECOVER()`

#### Description:

The function calls the Solidity `ecrecover()` function directly to verify the given signatures. However, the `ecrecover()` EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

References: https://swcregistry.io/docs/SWC-117, https://swcregistry.io/docs/SWC-121 and https://medium.com/cryptronics/signature-replay-vulnerabilities-in-smart-contracts-3b6f7596df57.

While this is not immediately exploitable, this may become a vulnerability if used elsewhere.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20PermitLight.sol

33:             address recoveredAddress = ecrecover(

```

#### Recommended Mitigation Steps:

Use the `ecrecover` function from OpenZeppelin’s ECDSA library for signature verification. (Ensure using a version > 4.7.3 for there was a critical bug >= 4.1.0 < 4.7.3).

### [NC-8] FUNCTIONS, PARAMETERS AND VARIABLES IN SNAKE CASE

#### Description:

Use camel case for all functions, parameters and variables and snake case for constants.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20PermitLight.sol

61:     function DOMAIN_SEPARATOR() public view returns (bytes32) {

```

### [NC-9] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES

#### Description:

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.

This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20.sol

14: import "./IERC20.sol";

15: import "./IERC677Receiver.sol";

```

```solidity
File: contracts/ERC20PermitLight.sol

7: import "./ERC20.sol";

```

```solidity
File: contracts/Equity.sol

6: import "./Frankencoin.sol";

7: import "./IERC677Receiver.sol";

8: import "./ERC20PermitLight.sol";

9: import "./MathUtil.sol";

10: import "./IReserve.sol";

```

```solidity
File: contracts/Frankencoin.sol

4: import "./ERC20PermitLight.sol";

5: import "./Equity.sol";

6: import "./IReserve.sol";

7: import "./IFrankencoin.sol";

```

```solidity
File: contracts/MintingHub.sol

4: import "./IERC20.sol";

5: import "./IReserve.sol";

6: import "./IFrankencoin.sol";

7: import "./Ownable.sol";

8: import "./IPosition.sol";

```

```solidity
File: contracts/Position.sol

4: import "./IERC20.sol";

5: import "./IPosition.sol";

6: import "./IReserve.sol";

7: import "./IFrankencoin.sol";

8: import "./Ownable.sol";

9: import "./MathUtil.sol";

```

```solidity
File: contracts/PositionFactory.sol

4: import "./Position.sol";

5: import "./IFrankencoin.sol";

```

```solidity
File: contracts/StablecoinBridge.sol

4: import "./IERC20.sol";

5: import "./IERC677Receiver.sol";

6: import "./IFrankencoin.sol";

```

#### Recommended Mitigation Steps:

`import {contract1 , contract2} from "filename.sol";` OR Use specific imports syntax per solidity docs recommendation.

### [NC-10] LARGE MULTIPLES OF TEN SHOULD USE SCIENTIFIC NOTATION

#### Description:

Use (e.g. 1e6) rather than decimal literals (e.g. 100000), for better code readability.

#### **Proof Of Concept**

```solidity
File: contracts/Equity.sol

211:         if (_votes * 10000 < QUORUM * totalVotes()) revert NotQualified();

```

```solidity
File: contracts/Frankencoin.sol

118:       return minterReserveE6 / 1000000;

205:       uint256 theoreticalReserve = _reservePPM * mintedAmount / 1000000;

239:       return 1000000 * amountExcludingReserve / (1000000 - adjustedReservePPM); // 41 / (1-18%) = 50

239:       return 1000000 * amountExcludingReserve / (1000000 - adjustedReservePPM); // 41 / (1-18%) = 50

```

```solidity
File: contracts/MathUtil.sol

11:     uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01

```

```solidity
File: contracts/MintingHub.sol

26:     uint32 public constant CHALLENGER_REWARD = 20000; // 2%

```

### [NC-11] FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION

#### Description:

https://blog.soliditylang.org/2022/03/16/solidity-0.8.13-release-announcement/

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

Use solidity pragma version min. 0.8.13

```solidity
File: contracts/ERC20.sol

12: pragma solidity ^0.8.0;

```

```solidity
File: contracts/ERC20PermitLight.sol

5: pragma solidity ^0.8.0;

```

```solidity
File: contracts/Equity.sol

4: pragma solidity >=0.8.0 <0.9.0;

```

```solidity
File: contracts/Frankencoin.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: contracts/MathUtil.sol

3: pragma solidity >=0.8.0 <0.9.0;

```

```solidity
File: contracts/MintingHub.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: contracts/Ownable.sol

9: pragma solidity ^0.8.0;

```

```solidity
File: contracts/Position.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: contracts/PositionFactory.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: contracts/StablecoinBridge.sol

2: pragma solidity ^0.8.0;

```

### [NC-12] ALLOWS MALLEABLE SECP256K1 SIGNATURES

#### Description:

Here, the `ecrecover()` method doesn’t check the s range.

Homestead (EIP-2) added this limitation, however the precompile remained unaltered. The majority of libraries, including OpenZeppelin, do this check.

Since an order can only be confirmed once and its hash is saved, there doesn’t seem to be a serious danger in existing use cases.

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/7201e6707f6631d9499a569f492870ebdd4133cf/contracts/utils/cryptography/ECDSA.sol#L138-L149

#### **Proof Of Concept**

```solidity
File: contracts/ERC20PermitLight.sol

33:             address recoveredAddress = ecrecover(
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

### [NC-13] MISSING EVENT FOR CRITICAL PARAMETER CHANGE

#### Description:

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

https://github.com/crytic/slither/wiki/Detector-Documentation#missing-events-access-control

#### **Proof Of Concept**

```solidity
File: contracts/ERC20.sol

158:         emit Transfer(sender, recipient, amount);

186:         emit Transfer(address(0), recipient, amount);

205:         emit Transfer(account, address(0), amount);

223:         emit Approval(owner, spender, value);

```

### [NC-14] MISSING FEE PARAMETER VALIDATION

#### Description:

Some fee parameters of functions are not checked for invalid values. Validate the parameters.

#### **Proof Of Concept**

```solidity
File: contracts/Frankencoin.sol

165:    function mint(address _target, uint256 _amount, uint32 _reservePPM, uint32 _feesPPM) override external minterOnly {

```

```solidity
File: contracts/MintingHub.sol

59:             function openPosition(

88:             function openPosition(

```

```solidity
File: contracts/Position.sol

50:             constructor(address _owner, address _hub, address _zchf, address _collateral,

```

```solidity
File: contracts/PositionFactory.sol

13:           function createNewPosition(address _owner, address _zchf, address _collateral, uint256 _minCollateral,

```

### [NC-15] NO SAME VALUE INPUT CONTROL

#### **Proof Of Concept**

```solidity
File: contracts/ERC20.sol

60:         decimals = _decimals;

```

```solidity
File: contracts/Frankencoin.sol

60:       MIN_APPLICATION_PERIOD = _minApplicationPeriod;

```

```solidity
File: contracts/Position.sol

56:         hub = _hub;

60:         mintingFeePPM = _mintingFeePPM;

63:         challengePeriod = _challengePeriod;

82:         limit = _limit;

```

### Recommended Mitigation Steps

Add code like this; if (oracle == \_oracle revert ADDRESS_SAME();

### [NC-16] FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE

#### Description:

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private, within a grouping, place the view and pure functions last

#### Context:

contracts/Position.sol
contracts/MintingHub.sol
contracts/Equity.sol
contracts/Frankencoin.sol
contracts/ERC20.sol
contracts/StablecoinBridge.sol

### [NC-17] USE A MORE RECENT VERSION OF SOLIDITY

#### Description:

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Proof Of Concept**

```solidity
File: contracts/ERC20.sol

12: pragma solidity ^0.8.0;

```

```solidity
File: contracts/ERC20PermitLight.sol

5: pragma solidity ^0.8.0;

```

```solidity
File: contracts/Equity.sol

4: pragma solidity >=0.8.0 <0.9.0;

```

```solidity
File: contracts/Frankencoin.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: contracts/MathUtil.sol

3: pragma solidity >=0.8.0 <0.9.0;

```

```solidity
File: contracts/MintingHub.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: contracts/Ownable.sol

9: pragma solidity ^0.8.0;

```

```solidity
File: contracts/Position.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: contracts/PositionFactory.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: contracts/StablecoinBridge.sol

2: pragma solidity ^0.8.0;

```

### [NC-18] Unused imports

#### **Proof Of Concept**

```solidity
File: contracts/Equity.sol

7: import "./IERC677Receiver.sol";

```

```solidity
File: contracts/Frankencoin.sol

6: import "./IReserve.sol";

```

```solidity
File: contracts/MintingHub.sol

4: import "./IERC20.sol";

5: import "./IReserve.sol";

7: import "./Ownable.sol";

```

```solidity
File: contracts/Position.sol

4: import "./IERC20.sol";

6: import "./IReserve.sol";

7: import "./IFrankencoin.sol";

```

```solidity
File: contracts/PositionFactory.sol

4: import "./Position.sol";

```

```solidity
File: contracts/StablecoinBridge.sol

4: import "./IERC20.sol";

5: import "./IERC677Receiver.sol";

```

### [NC-19] FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS

#### Description:

Solidity’s standard naming convention for internal and private functions and variables (apart from constants): the mixedCase format starting with an underscore (\_mixedCase starting with an underscore)

Solidity’s standard naming convention for internal and private constants variables: the snake_case format starting with an underscore (\_mixedCase starting with an underscore) and use ALL_CAPS for naming them.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20.so

97:     function allowanceInternal(address owner, address spender) internal view virtual returns (uint256) {

```

```solidity
File: contracts/Equity.sol

41:     uint256 private constant MINIMUM_EQUITY = 1000 * ONE_DEC18;

46:     uint32 private constant QUORUM = 300;

51:     uint8 private constant BLOCK_TIME_RESOLUTION_BITS = 24;

75:     uint192 private totalVotesAtAnchor;  // Total number of votes at the anchor time, see comment on the um

76:     uint64 private totalVotesAnchorTime; // 40 Bit for the block number, 24 Bit sub-block time resolution

88:     mapping (address => uint64) private voteAnchor; // 40 Bit for the block number, 24 Bit sub-block time resolution

144:     function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {

157:     function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){

172:     function anchorTime() internal view returns (uint64){

225:     function canVoteFor(address delegate, address owner) internal view returns (bool) {

266:     function calculateSharesInternal(uint256 capitalBefore, uint256 investment) internal view returns (uint256) {

```

```solidity
File: contracts/Frankencoin.sol

39:    uint256 private minterReserveE6;

102:    function allowanceInternal(address owner, address spender) internal view override returns (uint256) {

```

```solidity
File: contracts/MathUtil.sol

10:     uint256 internal constant ONE_DEC18 = 10**18;

11:     uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01

```

```solidity
File: contracts/MintingHub.sol

28:     IPositionFactory private immutable POSITION_FACTORY; // position contract to clone

188:     function minBid(Challenge storage challenge) internal view returns (uint256) {

287:     function returnCollateral(Challenge storage challenge, bool postpone) internal {

```

```solidity
File: contracts/Ownable.sol

39:     function setOwner(address newOwner) internal {

45:     function requireOwner(address sender) internal view {

```

```solidity
File: contracts/Position.sol

169:     function collateralBalance() internal view returns (uint256){

193:     function mintInternal(address target, uint256 amount, uint256 collateral_) internal {

202:     function restrictMinting(uint256 period) internal {

232:     function repayInternal(uint256 burnable) internal {

240:     function notifyRepaidInternal(uint256 amount) internal {

268:     function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {

282:     function checkCollateral(uint256 collateralReserve, uint256 atPrice) internal view {

286:     function emitUpdate() internal {

```

```solidity
File: contracts/PositionFactory.sol

37:     function createClone(address target) internal returns (address result) {

```

```solidity
File: contracts/StablecoinBridge.sol

49:     function mintInternal(address target, uint256 amount) internal {

67:     function burnInternal(address zchfHolder, address target, uint256 amount) internal {

```

## Low Issues

|      | Issue                                                                    |
| ---- | :----------------------------------------------------------------------- |
| L-1  | Arbitrary Jump with Function Type Variable                               |
| L-2  | DOS WITH BLOCK GAS LIMIT                                                 |
| L-3  | AVOID `TRANSFER()`/`SEND()` AS REENTRANCY MITIGATIONS                    |
| L-4  | AVOID USING LOW CALL FUNCTION ECRECOVER                                  |
| L-5  | CRITICAL ADDRESS CHANGES SHOULD USE TWO-STEP PROCEDURE                   |
| L-6  | DECODING AN IPFS HASH USING A FIXED HASH FUNCTION AND LENGTH OF THE HASH |
| L-7  | `ECRECOVER` MAY RETURN EMPTY ADDRESS                                     |
| L-8  | LOSS OF PRECISION DUE TO ROUNDING                                        |
| L-9  | OWNER CAN RENOUNCE OWNERSHIP                                             |
| L-10 | REQUIRE MESSAGES ARE TOO SHORT AND UNCLEAR                               |
| L-11 | Revert if ecrecover is address(0)                                        |
| L-12 | BLOCK VALUES AS A PROXY FOR TIME                                         |
| L-13 | UNSAFE CAST                                                              |
| L-14 | USE `_SAFEMINT` INSTEAD OF `_MINT`                                       |

### [L-1] Arbitrary Jump with Function Type Variable

#### Description:

Function types are supported in Solidity. This means that a variable of type function can be assigned to a function with a matching signature. The function can then be called from the variable just like any other function. Users should not be able to change the function variable, but in some cases this is possible.

If the smart contract uses certain assembly instructions, mstore for example, an attacker may be able to point the function variable to any other function. This may give the attacker the ability to break the functionality of the contract, and perhaps even drain the contract funds.

Since inline assembly is a way to access the EVM at a low level, it bypasses many important safety features. So it's important to only use assembly if it is necessary and properly understood.

[SWC Registry](https://swcregistry.io/docs/SWC-127)

#### **Proof Of Concept**

```solidity
File: contracts/PositionFactory.sol

39:         assembly {
                let clone := mload(0x40)
                mstore(clone, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
                mstore(add(clone, 0x14), targetBytes)
                mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
                result := create(0, clone, 0x37)
        }

```

### [L-2] DOS WITH BLOCK GAS LIMIT

#### Description:

When smart contracts are deployed or functions inside them are called, the execution of these actions always requires a certain amount of gas, based of how much computation is needed to complete them. The Ethereum network specifies a block gas limit and the sum of all transactions included in a block can not exceed the threshold.

Programming patterns that are harmless in centralized applications can lead to Denial of Service conditions in smart contracts when the cost of executing a function exceeds the block gas limit. Modifying an array of unknown size, that increases in size over time, can lead to such a Denial of Service condition.

[Code4arena example](https://swcregistry.io/docs/SWC-128)

#### **Proof Of Concept**

```solidity
File: contracts/Equity.sol

192:      for (uint i=0; i<helpers.length; i++){

196:      for (uint j=i+1; j<helpers.length; j++){

312:      for (uint256 i = 0; i<addressesToWipe.length; i++){

```

### [L-3] AVOID `TRANSFER()`/`SEND()` AS REENTRANCY MITIGATIONS

#### Description:

Although `transfer()` and `send()` have been recommended as a security best-practice to prevent reentrancy attacks because they only forward 2300 gas, the gas repricing of opcodes may break deployed contracts. Use `call()` instead, without hardcoded gas limits along with checks-effects-interactions pattern or reentrancy guards for reentrancy protection.

https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

https://swcregistry.io/docs/SWC-134

#### **Proof Of Concept**

```solidity
File: contracts/Equity.sol

279:         zchf.transfer(target, proceeds);

```

```solidity
File: contracts/MintingHub.sol

204:             zchf.transfer(challenge.bidder, challenge.bid); // return old bid

211:             challenge.position.collateral().transfer(msg.sender, challenge.size);

263:             IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);

268:             zchf.transfer(owner, effectiveBid - fundsNeeded);

272:         zchf.transfer(challenge.challenger, reward); // pay out the challenger reward

284:         IERC20(collateral).transfer(target, amount);

294:             challenge.position.collateral().transfer(challenge.challenger, challenge.size); // return the challenger's collateral

```

```solidity
File: contracts/Position.sol

253:             IERC20(token).transfer(target, amount);

269:         IERC20(collateral).transfer(target, amount);

```

```solidity
File: contracts/StablecoinBridge.sol

69:         chf.transfer(target, amount);

```

#### Recommended Mitigation Steps:

Using low-level `call.value(amount)` with the corresponding result check or using the OpenZeppelin `Address.sendValue` is advised:https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L60

### [L-4] AVOID USING LOW CALL FUNCTION ECRECOVER

#### Description:

Use OZ library ECDSA that its battle tested to avoid classic errors.

https://docs.openzeppelin.com/contracts/4.x/api/utils#ECDSA

#### **Proof Of Concept**

```solidity
File: contracts/ERC20PermitLight.sol

33:             address recoveredAddress = ecrecover(

```

### [L-5] CRITICAL ADDRESS CHANGES SHOULD USE TWO-STEP PROCEDURE

#### Description:

The critical procedures should be two step process.

Changing critical addresses in contracts should be a two-step process where the first transaction (from the old/current address) registers the new address (i.e. grants ownership) and the second transaction (from the new address) replaces the old address with the new one (i.e. claims ownership). This gives an opportunity to recover from incorrect addresses mistakenly used in the first step. If not, contract functionality might become inaccessible. (see [here](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1488) and [here](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/2369))

#### **Proof Of Concept**

```solidity
File: contracts/Ownable.sol

31:     function transferOwnership(address newOwner) public onlyOwner {

39:     function setOwner(address newOwner) internal {

```

#### Recommended Mitigation Steps:

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

### [L-6] DECODING AN IPFS HASH USING A FIXED HASH FUNCTION AND LENGTH OF THE HASH

#### Description:

An IPFS hash specifies the hash function and length of the hash in the first two bytes of the hash.

Although SHA256 is 32 bytes and is currently the most common IPFS hash function, other content could use a hash function that is larger than 32 bytes. The current implementation limits the usage to the SHA256 hash function and a hash length of 32 bytes.

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

Consider using a more generic implementation that can handle different hash functions and lengths and allow the user to choose.

```solidity
File: contracts/ERC20PermitLight.sol

41:                                 bytes32(0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9),

66:                     bytes32(0x47e79534a245952e8b16893a336b85a3d9ea9fa8c573f3d803afb92a79469218),

```

```solidity
File: contracts/PositionFactory.sol

41:             mstore(clone, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)

43:             mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)

43:             mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)

```

### [L-7] `ECRECOVER` MAY RETURN EMPTY ADDRESS

#### Description:

There is a common issue that ecrecover returns empty (0x0) address when the signature is invalid. function `_verifySigner` should check that before returning the result of ecrecover.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20PermitLight.sol

33:             address recoveredAddress = ecrecover(

```

#### Recommended Mitigation Steps:

See the solution here: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v3.4.0/contracts/cryptography/ECDSA.sol#L68

### [L-8] LOSS OF PRECISION DUE TO ROUNDING

#### **Proof Of Concept**

```solidity
File: contracts/Equity.sol

109:         return VALUATION_FACTOR * zchf.equity() * ONE_DEC18 / totalSupply();

```

```solidity
File: contracts/Frankencoin.sol

166:       uint256 usableMint = (_amount * (1000_000 - _feesPPM - _reservePPM)) / 1000_000; // rounding down is fine

205:       uint256 theoreticalReserve = _reservePPM * mintedAmount / 1000000;

209:          return theoreticalReserve * currentReserve / minterReserve();

239:       return 1000000 * amountExcludingReserve / (1000000 - adjustedReservePPM); // 41 / (1-18%) = 50

```

```solidity
File: contracts/MathUtil.sol

32:         return _a * _b / ONE_DEC18;

36:         return (_a * ONE_DEC18) / _b ;

```

```solidity
File: contracts/MintingHub.sol

165:             (challenge.bid * splitOffAmount) / challenge.size

189:         return (challenge.bid * 1005) / 1000;

265:         uint256 reward = (volume * CHALLENGER_REWARD) / 1000_000;

```

```solidity
File: contracts/Position.sol

80:         price = _mint * ONE_DEC18 / _coll;

122:             return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;

124:             return totalMint * (1000_000 - reserveContribution) / 1000_000;

187:             return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));

```

### [L-9] OWNER CAN RENOUNCE OWNERSHIP

#### Description:

Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The Openzeppelin’s Ownable used in this project contract implements `renounceOwnership`. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

#### **Proof Of Concept**

```solidity
File: contracts/MintingHub.sol

7: import "./Ownable.sol";

```

```solidity
File: contracts/Ownable.sol

19: contract Ownable {

```

```solidity
File: contracts/Position.sol

8: import "./Ownable.sol";

14: contract Position is Ownable, IPosition, MathUtil {

```

#### Recommended Mitigation Steps:

We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.

### [L-10] REQUIRE MESSAGES ARE TOO SHORT AND UNCLEAR

#### Description:

The correct and clear error description explains to the user why the function reverts, but the error descriptions below in the project are not self-explanatory. These error descriptions are very important in the debug features of DApps like Tenderly. Error definitions should be added to the require block, not exceeding 32 bytes.

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

Error definitions should be added to the require block, not exceeding 32 bytes or we should use custom errors

```solidity
File: contracts/ERC20.sol

152:         require(recipient != address(0));

180:         require(recipient != address(0));

```

```solidity
File: contracts/Equity.sol

194:             require(current != sender);

195:             require(canVoteFor(sender, current));

197:                 require(current != helpers[j]); // ensure helper unique

276:         require(canRedeem(msg.sender));

310:         require(zchf.equity() < MINIMUM_EQUITY);

```

```solidity
File: contracts/MintingHub.sol

158:         require(challenge.challenger != address(0x0));

171:         require(challenge.size >= min);

172:         require(copy.size >= min);

254:         require(challenge.challenger != address(0x0));


```

```solidity
File: contracts/Position.sol

53:         require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values

```

### [L-11] Revert if ecrecover is address(0)

#### Description:

On ERC20PermitLight.sol#L33 add a revert that triggers if the response is address(0), this means that signature its not valid.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20PermitLight.sol

33:             address recoveredAddress = ecrecover(

```

### [L-12] BLOCK VALUES AS A PROXY FOR TIME

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as block.timestamp, and block.number can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of block.timestamp, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers cant rely on the preciseness of the provided timestamp.

As for block.number, considering the block time on Ethereum is generally about 14 seconds, it`s possible to predict the time delta between blocks. However, block times are not constant and are subject to change for a variety of reasons, e.g. fork reorganisations and the difficulty bomb. Due to variable block times, block.number should also not be relied on for precise calculations of time.

Reference: https://swcregistry.io/docs/SWC-116

Reference: (https://github.com/kadenzipfel/smart-contract-vulnerabilities/blob/master/vulnerabilities/timestamp-dependence.md)

#### **Proof Of Concept**

```solidity
File: contracts/ERC20PermitLight.sol

30:         require(deadline >= block.timestamp, "PERMIT_DEADLINE_EXPIRED");

```

```solidity
File: contracts/Equity.sol

173:         return uint64(block.number << BLOCK_TIME_RESOLUTION_BITS);

```

```solidity
File: contracts/Frankencoin.sol

88:       minters[_minter] = block.timestamp + _applicationPeriod;

153:       if (block.timestamp > minters[_minter]) revert TooLate();

294:       return minters[_minter] != 0 && block.timestamp >= minters[_minter];

```

```solidity
File: contracts/MintingHub.sol

144:         challenges.push(Challenge(msg.sender, position, _collateralAmount, block.timestamp + position.challengePeriod(), address(0x0), 0));

201:         if (block.timestamp >= challenge.end) revert TooLate();

217:             uint256 earliestEnd = block.timestamp + 30 minutes;

240:         return challenges[_challengeNumber].end > block.timestamp;

255:         require(block.timestamp >= challenge.end, "period has not ended");

```

```solidity
File: contracts/Position.sol

64:         start = block.timestamp + initPeriod; // one week time to deny the position

110:         if (block.timestamp >= start) revert TooLate();

183:         uint256 time = block.timestamp;

203:         uint256 horizon = block.timestamp + period;

305:         if (block.timestamp >= expiration){

367:         if (block.timestamp > expiration) revert Expired();

374:         if (block.timestamp <= cooldown) revert Hot();

```

```solidity
File: contracts/StablecoinBridge.sol

29:         horizon = block.timestamp + 52 weeks;

50:         require(block.timestamp <= horizon, "expired");

```

### [L-13] UNSAFE CAST

#### Description:

Keep in mind that the version of solidity used, despite being greater than 0.8, does not prevent integer overflows during casting, it only does so in mathematical operations.

It is necessary to safely convert between the different numeric types.

#### **Proof Of Concept**

```solidity
File: contracts/Equity.sol

173:         return uint64(block.number << BLOCK_TIME_RESOLUTION_BITS);

```

```solidity
File: contracts/Position.sol

187:             return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));

```

#### Recommended Mitigation Steps:

Use a safeCast from Open Zeppelin or increase the type length.

### [L-14] USE `_SAFEMINT` INSTEAD OF `_MINT`

#### Description:

According to openzepplin’s ERC721, the use of `_mint` is discouraged, use `safeMint` whenever possible.

https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-mint-address-uint256-

#### **Proof Of Concept**

```solidity
File: contracts/ERC20.sol

179:     function _mint(address recipient, uint256 amount) internal virtual {

```

```solidity
File: contracts/Equity.sol

248:         _mint(from, shares);

```

```solidity
File: contracts/Frankencoin.sol

167:       _mint(_target, usableMint);

168:       _mint(address(reserve), _amount - usableMint); // rest goes to equity as reserves or as fees

173:       _mint(_target, _amount);

286:          _mint(msg.sender, _amount - reserveLeft);

```

#### Recommended Mitigation Steps:

Use `_safeMint` whenever possible instead of `_mint`
