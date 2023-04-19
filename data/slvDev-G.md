|  | Issue | Instances |
| --- | --- | --- |
| [G-01] | Pack state variables | 13 |
| [G-02] | Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate | 2 |
| [G-03] | Efficient Struct Packing to Save Storage Slots | 1 |
| [G-04] | Remove function canRedeem() | 1 |
| [G-05] | Improve For Loop Structure | 3 |
| [G-06] | Deploy the contract with clone instead of new (x10 Gas Saved) | 1 |
| [G-07] | Use nested if and, avoid multiple check combinations | 4 |
| [G-08] | Use hardcode address instead address(this) | 10 |
| [G-09] | Do not calculate constants variables | 6 |
| [G-10] | Multiple accesses of a mapping/array should use a local variable cache | 1 |
| [G-11] | Change public function visibility to external | 10 |
| [G-12] | Use assembly to write address storage values | 10 |
| [G-13] | Setting the constructor to payable (13 gas) | 5 |
| [G-14] | x += y (x -= y) costs more gas than x = x + y (x = x - y) for state variables | 12 |
| [G-15] | The solady Library's Ownable contract is significantly gas-optimized, which can be used | 1 |
| [G-16] | internal functions only called once can be inlined to save gas | 5 |
| [G-17] | Use Solmate SafeTransferLib  contracts | All Contracts |
| [G-18] | Functions guaranteed to revert when called by normal users can be marked payable | 6 |
| [G-19] | Optimize names to save gas (22 gas) | All Contracts |
| [G-20] | Sort Solidity operations using short-circuit mode | 1 |
| [G-21] | Use a more recent version of solidity | All Contracts |

## [G-01] **Pack state variables**

Solidity docs: [https://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html)

State variable packaging in `Equity.sol` contract (2 * 2k = 4k gas saved)

```solidity
File: contracts\Equity.sol

  uint32 public constant VALUATION_FACTOR = 3;
- uint256 private constant MINIMUM_EQUITY = 1000 * ONE_DEC18; 
// 1000 * ONE_DEC18 will require almost 70 bits, so we can use uint96
+ uint96 private constant MINIMUM_EQUITY = 1000 * ONE_DEC18; 
  uint32 private constant QUORUM = 300;
  uint8 private constant BLOCK_TIME_RESOLUTION_BITS = 24;
- uint256 public constant MIN_HOLDING_DURATION = 90*7200 << BLOCK_TIME_RESOLUTION_BITS;
// 90*7200 << BLOCK_TIME_RESOLUTION_BITS = 10,871,635,968,000
// smallest uint type that can accommodate is uint48
+  uint48 public constant MIN_HOLDING_DURATION = 90*7200 << BLOCK_TIME_RESOLUTION_BITS;

```

Combining variables that fit within one storage slot can save up to 2 slots in storage.

Additionally, you can change **`BLOCK_TIME_RESOLUTION_BITS`** to a larger uint type, such as **`uint32`**, and for **`MIN_HOLDING_DURATION`**, you can use **`uint64`**. These modifications will still allow the variables to be packed into a single storage slot.

State variable packaging in `Frankencoin.sol` contract (1 * 2k = 4k gas saved) or (2 * 2k = 4k gas saved)

```solidity
File: contracts\Frankencoin.sol

- uint256 public constant MIN_FEE = 1000 * (10**18);
- uint256 public immutable MIN_APPLICATION_PERIOD; // for example 10 days

// 1000 * (10**18) will require almost 70 bits, so we can use uint96
+ uint96 public constant MIN_FEE = 1000 * (10**18);
// uint160 for MIN_APPLICATION_PERIOD is more than enough
+ uint160 public immutable MIN_APPLICATION_PERIOD; // for example 10 days
// The maximum value that can be represented by a uint160 variable, when expressed in years
// is approximately ≈ 46,254,607,991,882,226,990 years

// OR

// additionaly if you are shure that minterReserveE6 can't be more than uint128
- uint256 private minterReserveE6;
// We can reduce MIN_APPLICATION_PERIOD to uint32 since
// maximum value that can be represented by a uint32 variable, when expressed in years, is approximately 136 years
+ uint32 public immutable MIN_APPLICATION_PERIOD; // for example 10 days
+ uint128 private minterReserveE6;
```

State variable packaging in `MathUtil.sol` contract (1 * 2k = 4k gas saved) 

No need to pack tighter since there are no other storage variables

```solidity
File: contracts\MathUtil.sol

- uint256 internal constant ONE_DEC18 = 10**18;
- uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01

+ uint128 internal constant ONE_DEC18 = 10**18;
+ uint128 internal constant THRESH_DEC18 =  10000000000000000;//0.01
```

State variable packaging in `MintingHub.sol` contract (1 * 2k = 4k gas saved) 

```solidity
File: contracts\MintingHub.sol

- uint256 public constant OPENING_FEE = 1000 * 10**18;
// 1000 * (10**18) will require almost 70 bits, so we can use uint96
+ uint96 public constant OPENING_FEE = 1000 * 10**18;
  uint32 public constant CHALLENGER_REWARD = 20000; // 2%
```

State variable packaging in `Position.sol` contract **(4 * 2k = 8k gas saved)** 

```solidity
File: contracts\Position.sol

uint256 public price; // the zchf price per unit of the collateral below which challenges succeed, (36 - collateral.decimals) decimals
uint256 public minted; // net minted amount, including reserve
uint256 public challengedAmount; // amount of the collateral that is currently under a challenge
+ uint256 public limit; // the minted amount must never exceed the limit
- uint256 public immutable challengePeriod; // challenge period in seconds

- uint256 public cooldown; // timestamp of the end of the latest cooldown
- uint256 public limit; // the minted amount must never exceed the limit

- uint256 public immutable start; // timestamp when minting can start
- uint256 public immutable expiration; // timestamp at which the position expires

// maximum value that can be represented by a uint32 variable, when expressed in years
// is approximately 136 years
+ uint32 public immutable challengePeriod; // challenge period in seconds

// uint32 timestamp will overflow on February 7, 2106
+ uint32 public cooldown; // timestamp of the end of the latest cooldown
+ uint32 public immutable start; // timestamp when minting can start
+ uint32 public immutable expiration; // timestamp at which the position expires

+ uint32 public immutable mintingFeePPM;
+ uint32 public immutable reserveContribution; // in ppm

// ^^ all uint32 will be packed in 192bits so you can increase some of them to uint64

address public immutable original; // originals point to themselves, clone to their origin
address public immutable hub; // the hub this position was created by
IFrankencoin public immutable zchf; // currency
IERC20 public override immutable collateral; // collateral
uint256 public override immutable minimumCollateral; // prevent dust amounts

- uint32 public immutable mintingFeePPM;
- uint32 public immutable reserveContribution; // in ppm
```

## [G-02] **Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, where appropriate**

Combining mappings into a single storage slot can lead to gas savings in certain situations, depending on the types and sizes of the variables involved. Merging mappings may help avoid a 20,000 gas cost (Gsset) for each combined mapping. Additionally, when a function requires both values and they fit into the same storage slot, read and write operations can become more efficient.

Moreover, if both fields are accessed within the same function, it can result in approximately 42 gas savings per access. This is because there's no need to recalculate the key's keccak256 hash (Gkeccak256, which costs 30 gas) and the associated stack operations for each field.

```solidity
File: contracts\Equity.sol

- mapping (address => address) public delegates;
- mapping (address => uint64) private voteAnchor;

+ struct VoterData {
+    address delegate;
+    uint64 voteAnchor;
+ }
+ mapping (address => VoterData) public voters;

```

```solidity
File: contracts\Frankencoin.sol

- mapping (address => uint256) public minters;
- mapping (address => address) public positions;

+ struct MinterData {
+   uint256 approvalTimeStamp; // Approval timestamp for the minter contract to mint Frankencoins
+   address position;          // Position that is allowed to mint and the minter that registered it
+ }

/**
 * Combined mapping for minters and positions.
 */
+ mapping (address => MinterData) public mintersAndPositions;
```

## [G-03] **Efficient Struct Packing to Save Storage Slots**

Reorganizing the **`Challenge`** struct can lead to fewer storage slots being used, which can save gas.

To save approximately 2k gas per slot, consider reorganizing the **`Challenge`** struct as follows

```solidity

struct Challenge {
    address challenger; // the address from which the challenge was initiated
    IPosition position; // the position that was challenged
    uint256 size;       // how much collateral the challenger provided
-   uint256 end;        // the deadline of the challenge (block.timestamp)
+   uint96 end;        // the deadline of the challenge (block.timestamp)
    address bidder;     // the address from which the highest bid was made, if any
    uint256 bid;        // the highest bid in ZCHF (total amount, not price per unit)
}
```

## [G-04] Remove function `canRedeem()`

Eliminate the **`canRedeem()`** function to save gas on deployment without causing issues.

Users can directly call **`canRedeem(address owner)`** with their address instead of using the **`canRedeem()`** function. This change will reduce gas consumption during deployment while maintaining functionality.

```solidity
File: contracts\Equity.sol
/**
 * Returns whether the sender address is allowed to redeem FPS.
 */
function canRedeem() external view returns (bool){
    return canRedeem(msg.sender);
}

/**
 * Returns whether the given address is allowed to redeem FPS, which is the
 * case after their average holding duration is larger than the required minimum.
 */
function canRedeem(address owner) public view returns (bool) {
    return anchorTime() - voteAnchor[owner] >= MIN_HOLDING_DURATION;
}
```

## [G-05] **Improve For Loop Structure**

Adjust the structure of for loops to reduce gas costs. This optimization can save approximately 400 gas for an array with 6 members.

```solidity
File: contracts\Equity.sol

192: for (uint i=0; i<helpers.length; i++){
196: for (uint j=i+1; j<helpers.length; j++){
312: for (uint256 i = 0; i<addressesToWipe.length; i++){
```

### Recommendation: Use Unchecked Increment and Cache Loop Length Locally

To achieve lower gas consumption in for loops, consider the following changes:

1. Use unchecked increment for the loop variable, as it can exceed the loop condition.
2. Cache the loop length locally to avoid repeatedly accessing the array length.

By implementing these changes, you can optimize for loops to be more gas-efficient while maintaining the desired functionality.

```solidity
+ uint256 loopLength = addressesToWipe.length;
- for (uint256 i = 0; i<addressesToWipe.length; i++){
+ for (uint256 i = 0; i<loopLength ;){
    address current = addressesToWipe[0];
    _burn(current, balanceOf(current));
+		unchecked {
+      i++;      
+   }
	}
```

## [G-06] Deploy the contract with `clone` instead of `new` (x10 Gas Saved)

Utilize OpenZeppelin Clones to save gas during deployment. Cloning contracts with **`Create2`** is more gas-efficient than using **`new`**.

**Recommendation: Implement OpenZeppelin Clones**

Leverage OpenZeppelin Clones to deploy minimal proxy contracts, reducing gas costs significantly.

**Comparison: Gas Usage**

A **`Create2`** clone is 10 times cheaper in gas usage compared to deploying with **`new`**.

[Gas usage difference between the two? (answer: a new clone is 10x cheaper)](https://github.com/porter-finance/v1-core/issues/15#issuecomment-1035639516)

```solidity
File: contracts\Frankencoin.sol

constructor(uint256 _minApplicationPeriod) ERC20(18){
  MIN_APPLICATION_PERIOD = _minApplicationPeriod;
  reserve = new Equity(this);
}
```

## [G-07] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier-to-read code and better coverage reports.

```solidity
File: contracts\Frankencoin.sol

if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();
```

```solidity
File: contracts\Position.sol

if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();
```

## [G-08] Use hardcode address instead `address(this)`

Instead of `address(this)`, it is more gas-efficient to pre-calculate and use the address with a hardcode. Foundry's `script.sol` and solmate `LibRlp.sol` contracts can do this.

Reference: [https://book.getfoundry.sh/reference/forge-std/compute-create-address](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

```solidity
File: contracts\MintingHub.sol

116: require(zchf.isPosition(position) == address(this), "not our pos");
142: IERC20(position.collateral()).transferFrom(msg.sender, address(this), _collateralAmount);
225: zchf.transferFrom(msg.sender, address(this), _bidAmountZCHF);
```

```solidity
File: contracts\Position.sol

55: original = address(this);
138: collateral.transferFrom(msg.sender, address(this), newCollateral - colbal);
170: return IERC20(collateral).balanceOf(address(this));
228: IERC20(zchf).transferFrom(msg.sender, address(this), amount);
```

```solidity
File: contracts\StablecoinBridge.sol

45: chf.transferFrom(msg.sender, address(this), amount);
51: require(chf.balanceOf(address(this)) <= limit, "limit");
79: burnInternal(address(this), from, amount);
```

## [G-09] **Do not calculate constants variables**

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.

```solidity
File: contracts\Equity.sol

uint256 private constant MINIMUM_EQUITY = 1000 * ONE_DEC18;
uint256 public constant MIN_HOLDING_DURATION = 90*7200 << BLOCK_TIME_RESOLUTION_BITS; // Set to 5 for local testing
```

```solidity
File: contracts\Frankencoin.sol

uint256 public constant MIN_FEE = 1000 * (10**18);
```

```solidity
File: contracts\MathUtil.sol

uint256 internal constant ONE_DEC18 = 10**18;
uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01
```

```solidity
File: contracts\MintingHub.sol

uint256 public constant OPENING_FEE = 1000 * 10**18;
```

## [G-10] **Multiple accesses of a mapping/array should use a local variable cache**

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata.

```solidity
File: contracts\Frankencoin.sol

function isMinter(address _minter) override public view returns (bool){
+  uint256 minter = minters[_minter];
+  return minter != 0 && block.timestamp >= minter;
-  return minters[_minter] != 0 && block.timestamp >= minters[_minter];
}
```

## [G-11] **Change `public` function visibility to `external`**

*Manually searched since the automatic tool has a lot of false positive findings.*

In contracts, certain functions are not called internally by other functions within the contract. To optimize gas usage, these functions can be made **`external`**.

Identify functions that are only called from outside the contract. Modify their visibility from **`public`** to **`external`**. This change will reduce gas consumption during external calls as **`external`** functions have a lower gas cost compared to **`public`** functions.

```solidity
File: contracts\Position.sol

function adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) public onlyOwner {
function repay(uint256 amount) public onlyOwner {
function deny(address[] calldata helpers, string calldata message) public {
function getUsableMint(uint256 totalMint, bool afterFees) public view returns (uint256){
function isClosed() public view returns (bool) {
```

```solidity
File: contracts\MintingHub.sol

function clonePosition(address position, uint256 _initialCollateral, uint256 _initialMint) public validPos(position) returns (address) {
```

```solidity
File: contracts\Frankencoin.sol

function equity() public view returns (uint256) {
```

```solidity
File: contracts\Equity.sol

function calculateShares(uint256 investment) public view returns (uint256) {
function redeem(address target, uint256 shares) public returns (uint256) {
function restructureCapTable(address[] calldata helpers, address[] calldata addressesToWipe) public {
```

## [G-12] Use `assembly` to write *address storage values*

```solidity
File: contracts\Equity.sol

constructor(Frankencoin zchf_) ERC20(18) {
    zchf = zchf_;
```

```solidity
File: contracts\Frankencoin.sol

constructor(uint256 _minApplicationPeriod) ERC20(18){
  reserve = new Equity(this);
```

```solidity
File: contracts\MintingHub.sol

constructor(address _zchf, address factory) {
    zchf = IFrankencoin(_zchf);
    POSITION_FACTORY = IPositionFactory(factory);
```

```solidity
File: 
constructor(...) {
        original = address(this);
        hub = _hub;
        zchf = IFrankencoin(_zchf);
        collateral = IERC20(_collateral);
```

```solidity
File: contracts\StablecoinBridge.sol

constructor(address other, address zchfAddress, uint256 limit_){
    chf = IERC20(other);
    zchf = IFrankencoin(zchfAddress);
```

## [G-13] **Setting the *constructor* to `payable` (13 gas)**

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of `msg.value == 0` and saves `13 gas` on deployments with no security risks.

```solidity
File: contracts\Equity.sol
constructor(Frankencoin zchf_) ERC20(18) {
```

```solidity
File: contracts\Frankencoin.sol
constructor(uint256 _minApplicationPeriod) ERC20(18){
```

```solidity
File: contracts\MintingHub.sol
constructor(address _zchf, address factory) {
```

```solidity
File: contracts\Position.sol
constructor(address _owner, address _hub, address _zchf, address _collateral,
```

```solidity
File: contracts\StablecoinBridge.sol
constructor(address other, address zchfAddress, uint256 limit_){
```

## [G-14] **`x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables**

```solidity
File: contracts\Frankencoin.sol

169: minterReserveE6 += _amount * _reservePPM;
196: minterReserveE6 -= amount * reservePPM;
217: minterReserveE6 -= targetTotalBurnAmount * _reservePPM;
253: minterReserveE6 -= freedAmount * _reservePPM;
```

```solidity
File: contracts\Position.sol

99: limit -= reduction + _minimum;
196: minted += amount;
242: minted -= amount;
295: challengedAmount += size;
309: challengedAmount -= _collateralAmount;
330: challengedAmount -= _size;
```

```solidity
File: contracts\ERC20.sol

184: _totalSupply += amount;
203: _totalSupply -= amount;
```

## [G-15] The solady Library's Ownable contract is significantly gas-optimized, which can be used

The project uses the `onlyOwner` authorization model I recommend using *Solady's highly gas optimized contract.*

[https://github.com/Vectorized/solady/blob/main/src/auth/OwnableRoles.sol](https://github.com/Vectorized/solady/blob/main/src/auth/OwnableRoles.sol)

```solidity
File: contracts\Position.sol

132: function adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) public onlyOwner {
159: function adjustPrice(uint256 newPrice) public onlyOwner noChallenge {
177: function mint(address target, uint256 amount) public onlyOwner noChallenge noCooldown alive {
227: function repay(uint256 amount) public onlyOwner {
249: function withdraw(address token, address target, uint256 amount) external onlyOwner {
263: function withdrawCollateral(address target, uint256 amount) public onlyOwner noChallenge noCooldown {
```

## [G-16] **`internal` functions only called once can be inlined to save gas**

Not inlining costs **20** to **40** gas because of two extra `JUMP` instructions and additional stack operations needed for function calls.

```solidity
File: contracts\MintingHub.sol
function returnCollateral(Challenge storage challenge, bool postpone) internal {
```

```solidity
File: contracts\PositionFactory.sol
function createClone(address target) internal returns (address result) {
```

```solidity
File: contracts\Equity.sol

function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {
function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){
```

```solidity
File: contracts\Position.sol
function repayInternal(uint256 burnable) internal {
```

## [G-17] Use `Solmate SafeTransferLib`  contracts

Use the gas-optimized Solmate SafeTransferLib contract for Erc20

[https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol](https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol)

## [G-18] **Functions guaranteed to *revert* when called by normal users can be marked `payable`**

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

```solidity
File: contracts\Position.sol

function adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) public onlyOwner {
function adjustPrice(uint256 newPrice) public onlyOwner noChallenge {
function mint(address target, uint256 amount) public onlyOwner noChallenge noCooldown alive {
function repay(uint256 amount) public onlyOwner {
function withdraw(address token, address target, uint256 amount) external onlyOwner {
function withdrawCollateral(address target, uint256 amount) public onlyOwner noChallenge noCooldown {
```

## [G-19] Optimize names to save gas (22 gas)

Contracts most called functions could simply save gas by function ordering via `Method ID`. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because `22 gas` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

**Context:**  All Contracts

**Recommendation:**  Find a lower `method ID` name for the most called functions for example Call() vs. Call1() is cheaper by `22 gas` For example, the function IDs in the `Erc20Quest.sol` contract will be the most used; A lower method ID may be given.

**Proof of Consept:** [https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

## [G-20] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses `OR/AND` logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```solidity
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 

//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```

```solidity
File: contracts\Frankencoin.sol

} else if (isMinter(spender) || isMinter(isPosition(spender))){
```

## [G-21] **Use a more recent version of solidity**

All contracts included in the scope has unspecified pragma `solidity ^0.8.0`. I recommend that you upgrade the versions of all contracts in scope to the latest version of robustness, '0.8.17’.

> Solidity 0.8.10 has a useful change that reduced gas costs of external calls which expect a return value.
> 

> In 0.8.15 the conditions necessary for inlining are relaxed. Benchmarks show that the change significantly decreases the bytecode size (which impacts the deployment cost) while the effect on the runtime gas usage is smaller.
> 

> In 0.8.17 prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call; Simplify the starting offset of zero-length operations to zero. More efficient overflow checks for multiplication.
>