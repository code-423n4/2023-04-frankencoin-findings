## Gas Optimizations

|        | Issue                                                                       |
| ------ | :-------------------------------------------------------------------------- |
| GAS-1  | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE |
| GAS-2  | SETTING THE CONSTRUCTOR TO PAYABLE                                          |
| GAS-3  | DOS WITH BLOCK GAS LIMIT                                                    |
| GAS-4  | USE FUNCTION INSTEAD OF MODIFIERS                                           |
| GAS-5  | MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT           |
| GAS-6  | THE INCREMENT IN FOR LOOP POSTCONDITION CAN BE MADE UNCHECKED               |
| GAS-7  | PROPER DATA TYPES                                                           |
| GAS-8  | USING SOLIDITY VERSION 0.8.19 WILL PROVIDE AN OVERALL GAS OPTIMIZATION      |
| GAS-9  | TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT                         |
| GAS-10 | USAGE OF `UINT`/`INT` SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD      |
| GAS-11 | UNNECESSARY LIBRARIES                                                       |
| GAS-12 | USE BYTES32 INSTEAD OF STRING                                               |

### [GAS-1] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

#### Description:

Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20.sol

85:     function transfer(address recipient, uint256 amount) public virtual override returns (bool) {

125:     function transferFrom(address sender, address recipient, uint256 amount) external override returns (bool) {

151:     function _transfer(address sender, address recipient, uint256 amount) internal virtual {

162:     function transferAndCall(address recipient, uint256 amount, bytes calldata data) external override returns (bool) {

```

```solidity
File: contracts/Equity.sol

279:         zchf.transfer(target, proceeds);

```

```solidity
File: contracts/MintingHub.sol

108:         zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE);

110:         IERC20(_collateralAddress).transferFrom(msg.sender, address(pos), _initialCollateral);

129:         existing.collateral().transferFrom(msg.sender, address(pos), _initialCollateral);

142:         IERC20(position.collateral()).transferFrom(msg.sender, address(this), _collateralAmount);

204:             zchf.transfer(challenge.bidder, challenge.bid); // return old bid

210:             zchf.transferFrom(msg.sender, challenge.challenger, _bidAmountZCHF);

211:             challenge.position.collateral().transfer(msg.sender, challenge.size);

225:             zchf.transferFrom(msg.sender, address(this), _bidAmountZCHF);

263:             IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);

268:             zchf.transfer(owner, effectiveBid - fundsNeeded);

272:         zchf.transfer(challenge.challenger, reward); // pay out the challenger reward

284:         IERC20(collateral).transfer(target, amount);

294:             challenge.position.collateral().transfer(challenge.challenger, challenge.size); // return the challenger's collateral

```

```solidity
File: contracts/Position.sol

138:             collateral.transferFrom(msg.sender, address(this), newCollateral - colbal);

228:         IERC20(zchf).transferFrom(msg.sender, address(this), amount);

253:             IERC20(token).transfer(target, amount);

269:         IERC20(collateral).transfer(target, amount);

352:         internalWithdrawCollateral(_bidder, _size); // transfer collateral to the bidder and emit update

```

```solidity
File: contracts/StablecoinBridge.sol

45:         chf.transferFrom(msg.sender, address(this), amount);

69:         chf.transfer(target, amount);

```

### [GAS-2] SETTING THE CONSTRUCTOR TO PAYABLE

#### Description:

Saves ~13 gas per instance

#### **Proof Of Concept**

```solidity
File: contracts/ERC20.sol

59:     constructor(uint8 _decimals) {

```

```solidity
File: contracts/Equity.sol

93:     constructor(Frankencoin zchf_) ERC20(18) {

```

```solidity
File: contracts/Frankencoin.sol

59:    constructor(uint256 _minApplicationPeriod) ERC20(18){

```

```solidity
File: contracts/MintingHub.sol

54:     constructor(address _zchf, address factory) {

```

```solidity
File: contracts/Position.sol

50:     constructor(address _owner, address _hub, address _zchf, address _collateral,

```

```solidity
File: contracts/StablecoinBridge.sol

26:     constructor(address other, address zchfAddress, uint256 limit_){

```

### [GAS-3] DOS WITH BLOCK GAS LIMIT

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

#### Recommended Mitigation Steps:

Caution is advised when you expect to have large arrays that grow over time. Actions that require looping across the entire data structure should be avoided.

If you absolutely must loop over an array of unknown size, then you should plan for it to potentially take multiple blocks, and therefore require multiple transactions.

### [GAS-4] USE FUNCTION INSTEAD OF MODIFIERS

#### **Proof Of Concept**

```solidity
File: contracts/Frankencoin.sol

266:    modifier minterOnly() {

```

```solidity
File: contracts/MintingHub.sol

115:     modifier validPos(address position) {

```

```solidity
File: contracts/Ownable.sol

49:     modifier onlyOwner() {

```

```solidity
File: contracts/Position.sol

366:     modifier alive() {

373:     modifier noCooldown() {

380:     modifier noChallenge() {

387:     modifier onlyHub() {

```

### [GAS-5] MAKING CONSTANT VARIABLES PRIVATE WILL SAVE GAS DURING DEPLOYMENT

#### Description:

When constants are marked public, extra getter functions are created, increasing the deployment cost. Marking these functions private will decrease gas cost. One can still read these variables through the source code. If they need to be accessed by an external contract, a separate single getter function can be used to return all constants as a tuple. There are four instances of public constants.

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

### [GAS-6] THE INCREMENT IN FOR LOOP POSTCONDITION CAN BE MADE UNCHECKED

#### Description:

This is only relevant if you are using the default solidity checked arithmetic.

The for loop postcondition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of i to enter the for-loop body is `2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks.One can manually do this.

[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

```solidity
File: contracts/Equity.sol

192:         for (uint i=0; i<helpers.length; i++){

196:             for (uint j=i+1; j<helpers.length; j++){

312:         for (uint256 i = 0; i<addressesToWipe.length; i++){

```

### [GAS-7] PROPER DATA TYPES

#### Description:

In Solidity, some data types are more expensive than others. It’s important to be aware of the most efficient type that can be used. Here are a few rules about data types.

Type uint should be used in place of type string whenever possible.

Type uint256 takes less gas to store than uint8

Type bytes should be used over byte[]

If the length of bytes can be limited, use the lowest amount possible from bytes1 to bytes32.

Type bytes32 is cheaper to use than type string and bytes.

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

[Source](https://betterprogramming.pub/how-to-write-smart-contracts-that-optimize-gas-spent-on-ethereum-30b5e9c5db85)

Fixed size variables are always cheaper than dynamic ones.

[Source](https://medium.com/coinmonks/gas-optimization-in-solidity-part-i-variables-9d5775e43dde)

Most of the time it will be better to use a mapping instead of an array because of its cheaper operations.

#### **Proof Of Concept**

```solidity
File: contracts/ERC20.sol

51:     uint8 public immutable override decimals;

59:     constructor(uint8 _decimals) {

```

```solidity
File: contracts/ERC20PermitLight.sol

26:         uint8 v,

```

```solidity
File: contracts/Equity.sol

39:     uint32 public constant VALUATION_FACTOR = 3;

46:     uint32 private constant QUORUM = 300;

51:     uint8 private constant BLOCK_TIME_RESOLUTION_BITS = 24;

75:     uint192 private totalVotesAtAnchor;  // Total number of votes at the anchor time, see comment on the um

76:     uint64 private totalVotesAnchorTime; // 40 Bit for the block number, 24 Bit sub-block time resolution

88:     mapping (address => uint64) private voteAnchor; // 40 Bit for the block number, 24 Bit sub-block time resolution

161:             voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); // new example anchor is only 21 / 11 = 1 block in the past

172:     function anchorTime() internal view returns (uint64){

173:         return uint64(block.number << BLOCK_TIME_RESOLUTION_BITS);

192:         for (uint i=0; i<helpers.length; i++){

196:             for (uint j=i+1; j<helpers.length; j++){

```

```solidity
File: contracts/Frankencoin.sol

165:    function mint(address _target, uint256 _amount, uint32 _reservePPM, uint32 _feesPPM) override external minterOnly {

204:    function calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) public view returns (uint256) {

223:    function burnFrom(address payer, uint256 targetTotalBurnAmount, uint32 _reservePPM) external override minterOnly returns (uint256) {

251:    function burnWithReserve(uint256 _amountExcludingReserve, uint32 _reservePPM) external override minterOnly returns (uint256) {

```

```solidity
File: contracts/MintingHub.sol

26:     uint32 public constant CHALLENGER_REWARD = 20000; // 2%

91:         uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address) {

309:         uint32 _mintingFeePPM,

311:         uint32 _reserve

```

```solidity
File: contracts/Position.sol

38:     uint32 public immutable mintingFeePPM;

39:     uint32 public immutable reserveContribution; // in ppm

187:             return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));

```

```solidity
File: contracts/PositionFactory.sol

15:         uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reserve)

38:         bytes20 targetBytes = bytes20(target);

```

### [GAS-8] USING SOLIDITY VERSION 0.8.19 WILL PROVIDE AN OVERALL GAS OPTIMIZATION

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

### [GAS-9] TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT

#### Description:

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

#### **Proof Of Concept**

```solidity
File: contracts/Frankencoin.sol

141:  if (balance <= minReserve){
        return 0;
      } else {
        return balance - minReserve;
      }

207:   if (currentReserve < minterReserve()){
         // not enough reserves, owner has to take a loss
         return theoreticalReserve * currentReserve / minterReserve();
      } else {
         return theoreticalReserve;
      }

```

```solidity
File: contracts/Position.sol

121:    if (afterFees){
            return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
        } else {
            return totalMint * (1000_000 - reserveContribution) / 1000_000;
        }

160:    if (newPrice > price) {
            restrictMinting(3 days);
        } else {
            checkCollateral(collateralBalance(), newPrice);
        }

184:    if (time >= exp){
            return 0;
        } else {
            return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));
        }

250:    if (token == address(collateral)){
            withdrawCollateral(target, amount);
        } else {
            IERC20(token).transfer(target, amount);
        }

```

### [GAS-10] USAGE OF `UINT`/`INT` SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD

#### Description:

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

#### **Proof Of Concept**

```solidity
File: contracts/ERC20.sol

51:     uint8 public immutable override decimals;

59:     constructor(uint8 _decimals) {

```

```solidity
File: contracts/ERC20PermitLight.sol

26:         uint8 v,

```

```solidity
File: contracts/Equity.sol

39:     uint32 public constant VALUATION_FACTOR = 3;

46:     uint32 private constant QUORUM = 300;

51:     uint8 private constant BLOCK_TIME_RESOLUTION_BITS = 24;

76:     uint64 private totalVotesAnchorTime; // 40 Bit for the block number, 24 Bit sub-block time resolution

88:     mapping (address => uint64) private voteAnchor; // 40 Bit for the block number, 24 Bit sub-block time resolution

161:             voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); // new example anchor is only 21 / 11 = 1 block in the past

172:     function anchorTime() internal view returns (uint64){

173:         return uint64(block.number << BLOCK_TIME_RESOLUTION_BITS);

```

```solidity
File: contracts/Frankencoin.sol

165:    function mint(address _target, uint256 _amount, uint32 _reservePPM, uint32 _feesPPM) override external minterOnly {

194:    function burn(uint256 amount, uint32 reservePPM) external override minterOnly {

204:    function calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) public view returns (uint256) {

223:    function burnFrom(address payer, uint256 targetTotalBurnAmount, uint32 _reservePPM) external override minterOnly returns (uint256) {

235:    function calculateFreedAmount(uint256 amountExcludingReserve /* 41 */, uint32 reservePPM /* 20% */) public view returns (uint256){

251:    function burnWithReserve(uint256 _amountExcludingReserve, uint32 _reservePPM) external override minterOnly returns (uint256) {

```

```solidity
File: contracts/MintingHub.sol

26:     uint32 public constant CHALLENGER_REWARD = 20000; // 2%

62:         uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address) {

91:         uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address) {

260:         (address owner, uint256 effectiveBid, uint256 volume, uint256 repayment, uint32 reservePPM) = challenge.position.notifyChallengeSucceeded(recipient, challenge.bid, challenge.size);

309:         uint32 _mintingFeePPM,

311:         uint32 _reserve

```

```solidity
File: contracts/Position.sol

38:     uint32 public immutable mintingFeePPM;

39:     uint32 public immutable reserveContribution; // in ppm

52:         uint256 _challengePeriod, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) {

181:     function calculateCurrentFee() public view returns (uint32) {

187:             return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));

329:     function notifyChallengeSucceeded(address _bidder, uint256 _bid, uint256 _size) external onlyHub returns (address, uint256, uint256, uint256, uint32) {

```

```solidity
File: contracts/PositionFactory.sol

15:         uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reserve)

```

### [GAS-11] UNNECESSARY LIBRARIES

#### Description:

Libraries are often only imported for a small number of uses, meaning that they can contain a significant amount of code that is redundant to your contract. If you can safely and effectively implement the functionality imported from a library within your contract, it is optimal to do so.
[Source](https://betterprogramming.pub/how-to-write-smart-contracts-that-optimize-gas-spent-on-ethereum-30b5e9c5db85)

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

### [GAS-12] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: contracts/Equity.sol

97:     function name() override external pure returns (string memory) {

101:     function symbol() override external pure returns (string memory) {

```

```solidity
File: contracts/Frankencoin.sol

52:    event MinterApplied(address indexed minter, uint256 applicationPeriod, uint256 applicationFee, string message);

53:    event MinterDenied(address indexed minter, string message);

64:    function name() override external pure returns (string memory){

68:    function symbol() override external pure returns (string memory){

83:    function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {

152:    function denyMinter(address _minter, address[] calldata _helpers, string calldata _message) override external {

```

```solidity
File: contracts/Position.sol

43:     event PositionDenied(address indexed sender, string message); // emitted if closed by governance

109:     function deny(address[] calldata helpers, string calldata message) public {

```
