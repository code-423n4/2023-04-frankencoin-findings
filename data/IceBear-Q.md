# Frankencoin
## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [N-1](#N-1) | Use of block.timestamp | 19 |
| [N-2](#N-2) | Return values of `approve()` not checked  | 3 |
| [N-3](#N-3) | Unused parameters | 3 |
| [N-4](#N-4) | Functions not used internally could be marked external | 14 |
| [N-5](#N-5) | Constant values such as a call to keccak256(), should used to immutable rather than constant | 6 |
| [N-6](#N-6) | Assembly Codes Specific – Should Have Comments | 1 |
| [N-7](#N-7) | Lack of events emission after sensitive actions | 5 |

### [N-1] Use of block.timestamp
Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.
#### Recommended Mitigation Steps
Block timestamps should not be used for entropy or generating random numbers — i.e., they should not be the deciding factor (either directly or through some derivation) for winning a game or changing an important state.

Time-sensitive logic is sometimes required; e.g., for unlocking contracts (time-locking), completing an ICO after a few weeks, or enforcing expiry dates. It is sometimes recommended to use block.number and an average block time to estimate times; with a 10 second block time, 1 week equates to approximately, 60480 blocks. Thus, specifying a block number at which to change a contract state can be more secure, as miners are unable to easily manipulate the block number.

Instances where block.timestamp is used:

*Find (19) instance(s) in contracts*:
```solidity
File: ERC20PermitLight.sol

30:         require(deadline >= block.timestamp, "PERMIT_DEADLINE_EXPIRED");

```
[ERC20PermitLight.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol)

```solidity
File: Frankencoin.sol

88:       minters[_minter] = block.timestamp + _applicationPeriod;

153:       if (block.timestamp > minters[_minter]) revert TooLate();

294:       return minters[_minter] != 0 && block.timestamp >= minters[_minter];

```
[Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol)

```solidity
File: MintingHub.sol

43:         uint256 end;        // the deadline of the challenge (block.timestamp)

144:         challenges.push(Challenge(msg.sender, position, _collateralAmount, block.timestamp + position.challengePeriod(), address(0x0), 0));

201:         if (block.timestamp >= challenge.end) revert TooLate();

217:             uint256 earliestEnd = block.timestamp + 30 minutes;

240:         return challenges[_challengeNumber].end > block.timestamp;

255:         require(block.timestamp >= challenge.end, "period has not ended");

```
[MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol)

```solidity
File: Position.sol

64:         start = block.timestamp + initPeriod; // one week time to deny the position

110:         if (block.timestamp >= start) revert TooLate();

183:         uint256 time = block.timestamp;

203:         uint256 horizon = block.timestamp + period;

305:         if (block.timestamp >= expiration){

367:         if (block.timestamp > expiration) revert Expired();

374:         if (block.timestamp <= cooldown) revert Hot();

```
[Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol)

```solidity
File: StablecoinBridge.sol

29:         horizon = block.timestamp + 52 weeks;

50:         require(block.timestamp <= horizon, "expired");

```
[StablecoinBridge.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol)


### [N-2] Return values of `approve()` not checked 
Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

*Find (3) instance(s) in contracts*:
```solidity
File: ERC20.sol

109:         _approve(msg.sender, spender, value);

132:             _approve(sender, msg.sender, currentAllowance - amount);

```
[ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol)

```solidity
File: ERC20PermitLight.sol

57:             _approve(recoveredAddress, spender, value);

```
[ERC20PermitLight.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol)


### [N-3] Unused parameters

*Find (3) instance(s) in contracts*:
```solidity
File: ERC20.sol

240:     function _beforeTokenTransfer(address from, address to, uint256 amount) virtual internal {

```
[ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol)



### [N-4] Functions not used internally could be marked external 

*Find (14) instance(s) in contracts*:
```solidity
File: ERC20.sol

66:     function totalSupply() public view override returns (uint256) {

73:     function balanceOf(address account) public view override returns (uint256) {

```
[ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol)

```solidity
File: ERC20PermitLight.sol

21:     function permit(

```
[ERC20PermitLight.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol)

```solidity
File: Equity.sol

262:     function calculateShares(uint256 investment) public view returns (uint256) {

275:     function redeem(address target, uint256 shares) public returns (uint256) {

309:     function restructureCapTable(address[] calldata helpers, address[] calldata addressesToWipe) public {

```
[Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol)

```solidity
File: Frankencoin.sol

138:    function equity() public view returns (uint256) {

```
[Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol)

```solidity
File: MintingHub.sol

124:     function clonePosition(address position, uint256 _initialCollateral, uint256 _initialMint) public validPos(position) returns (address) {

```
[MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol)

```solidity
File: Ownable.sol

31:     function transferOwnership(address newOwner) public onlyOwner {

```
[Ownable.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol)

```solidity
File: Position.sol

109:     function deny(address[] calldata helpers, string calldata message) public {

120:     function getUsableMint(uint256 totalMint, bool afterFees) public view returns (uint256){

132:     function adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) public onlyOwner {

227:     function repay(uint256 amount) public onlyOwner {

360:     function isClosed() public view returns (bool) {

```
[Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol)

### [N-5] Constant values such as a call to keccak256(), should used to immutable rather than constant
There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.
  
Constants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

*Find (6) instance(s) in contracts*:
```solidity
File: ERC20.sol

47:     uint256 internal constant INFINITY = (1 << 255);

```
[ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol)

```solidity
File: Equity.sol

41:     uint256 private constant MINIMUM_EQUITY = 1000 * ONE_DEC18;

59:     uint256 public constant MIN_HOLDING_DURATION = 90*7200 << BLOCK_TIME_RESOLUTION_BITS; // Set to 5 for local testing

```
[Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol)

```solidity
File: Frankencoin.sol

25:    uint256 public constant MIN_FEE = 1000 * (10**18);

```
[Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol)

```solidity
File: MathUtil.sol

10:     uint256 internal constant ONE_DEC18 = 10**18;

```
[MathUtil.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol)

```solidity
File: MintingHub.sol

20:     uint256 public constant OPENING_FEE = 1000 * 10**18;

```
[MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol)

### [N-6]Assembly Codes Specific – Should Have Comments
Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Aseembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

```solidity
File: PositionFactory.sol

39:     assembly {
40:            let clone := mload(0x40)
41:            mstore(clone, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
42:            mstore(add(clone, 0x14), targetBytes)
43:           mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
44:            result := create(0, clone, 0x37)
45:        }

```
[PositionFactory.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol)

### [N-7]Lack of events emission after sensitive actions

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.
similar findings:
- https://blog.openzeppelin.com/holdefi-audit/#medium
- https://code4rena.com/reports/2022-12-pooltogether

```solidity
File: StablecoinBridge.sol

44:   function mint(address target, uint256 amount) public {
    
63:   function burn(address target, uint256 amount) external {
    
75:   function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool){
```
[StablecoinBridge.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol)

```solidity
File: Position.sol

249:  function withdraw(address token, address target, uint256 amount) external onlyOwner {
    
249:  function withdrawCollateral(address target, uint256 amount) public onlyOwner noChallenge noCooldown {

```
[Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol)

#### Recommendation
Add Event-Emit.

## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Owner can renounce ownership | 7 |
| [L-2](#L-2) | Minting tokens to the zero address should be avoided | 1 |


### [L-1] Owner can renounce ownership
Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.
    
Ownable used in this project contract implements renounceOwnership. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.
    
#### Recommendation
We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.

*Find (7) instance(s) in contracts*:
```solidity
File: Position.sol

8: import "./Ownable.sol";

132:     function adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) public onlyOwner {

159:     function adjustPrice(uint256 newPrice) public onlyOwner noChallenge {

177:     function mint(address target, uint256 amount) public onlyOwner noChallenge noCooldown alive {

227:     function repay(uint256 amount) public onlyOwner {

249:     function withdraw(address token, address target, uint256 amount) external onlyOwner {

263:     function withdrawCollateral(address target, uint256 amount) public onlyOwner noChallenge noCooldown {

```
[Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol)

### [L-2]  Minting tokens to the zero address should be avoided
Consider applying a check in the function to ensure tokens aren’t minted to the zero address.

```solidity
File: StablecoinBridge.sol

44:   function mint(address target, uint256 amount) public {

```
[StablecoinBridge.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol)

