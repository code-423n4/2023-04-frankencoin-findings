## Missing cancel option on bid() and end()
Consider adding checks on [`bid()`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L199-L229) and [`end()`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L252-L276) such that the functions will have `challenges[_challengeNumber]` and `challenge.position.challengedAmount` deleted when `challenge.position.collateral() == 0` while returning `challenge.bid` to `challenge.bidder`. This early termination is going to save a huge amount of gas making all futile zero transfers.

## Unneeded instant cast
Casting an instant to a contract type is unnecessary and is a waste of gas. Do this only when you are casting the contract address.

Here are some of the instances entailed:

[File: Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol)

```solidity
111:        IReserve(zchf.reserve()).checkQualified(msg.sender, helpers);

170:        return IERC20(collateral).balanceOf(address(this));

228:        IERC20(zchf).transferFrom(msg.sender, address(this), amount);

209:        IERC20(collateral).transfer(target, amount);
```
[File: MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol)

```solidity
142:        IERC20(position.collateral()).transferFrom(msg.sender, address(this), _collateralAmount);

263:            IERC20(zchf).transfer(challenge.bidder, challenge.bid - effectiveBid);

284:        IERC20(collateral).transfer(target, amount);
```
## State Variables Repeatedly Read Should be Cached
SLOADs cost 100 gas each after the 1st one whereas MLOADs/MSTOREs only incur 3 gas each. As such, storage values read multiple times should be cached in the stack memory the first time (costing only 1 SLOAD) and then re-read from this cache to avoid multiple SLOADs.

For instance, `mintingFeePPM` and `start` in the code block below should be cached as follows:

[File: Position.sol#L187](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L187)

```diff
+            uint32 _mintingFeePPM = mintingFeePPM; // SLOAD 1
+            uint256 _start = start; // SLOAD 1
-            return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));
+            // MLOAD 1, 2, 4 & 4
+            return uint32(_mintingFeePPM - _mintingFeePPM * (time - _start) / (exp - _start));
```
## Unneeded global variable cache
There is negligible gas benefit caching a global variable unless it is entailed in a considerably large loop.

Here is a specific instance entailed:

[File: Position.sol#L183](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L183)

```solidity
        uint256 time = block.timestamp;
```
## Non-strict inequalities are cheaper than strict ones
In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).

As an example, consider replacing `>=` with the strict counterpart `>` in the following inequality instance:

[File: Equity.sol#L136](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L136)

```diff
-        return anchorTime() - voteAnchor[owner] >= MIN_HOLDING_DURATION;
        // Rationale for subtracting 1 on the right side of the inequality:
        // x >= 10; [10, 11, 12, ...]
        // x > 10 - 1 is the same as x > 9; [10, 11, 12 ...]
+        return anchorTime() - voteAnchor[owner] > MIN_HOLDING_DURATION - 1;
```
Similarly, the `<=` instance below may be replaced with `<` as follows:

[File: Frankencoin.sol#L141](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L141)

```diff
-      if (balance <= minReserve){
+      if (balance < minReserve + 1){
```
## `||` costs less gas than its equivalent `&&`
Rule of thumb: `(x && y)` is `(!(!x || !y))`

Even with the 10k Optimizer enabled: `||`, OR conditions cost less than their equivalent `&&`, AND conditions.

As an example, the instance below may be refactored as follows:

[File: Frankencoin.sol#L267](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L267)

```diff
-      if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();
+      if (!(isMinter(msg.sender) || isMinter(positions[msg.sender]))) revert NotMinter();
```
## Private function with embedded modifier reduces contract size
Consider having the logic of a modifier embedded through a private function to reduce contract size if need be. A `private` visibility that saves more gas on function calls than the `internal` visibility is adopted because the modifier will only be making this call inside the contract.

For example, the modifier instance below may be refactored as follows:

[File: Position.sol#L373-L376](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L373-L376)

```diff
+    function _noCooldown() private view {
+        if (block.timestamp <= cooldown) revert Hot();
+    }

    modifier noCooldown() {
-        if (block.timestamp <= cooldown) revert Hot();
+        _noCooldown();
        _;
    }
```
## Use of named returns for local variables saves gas
You can have further advantages in term of gas cost by simply using named return values as temporary local variable.

For instance, the code block below may be refactored as follows:

[File: Position.sol#L268-L276](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L268-L276)

```diff
-    function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {
+    function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256 balance) {
        IERC20(collateral).transfer(target, amount);
-        uint256 balance = collateralBalance();
+        balance = collateralBalance();
        if (balance < minimumCollateral){
            cooldown = expiration;
        }
        emitUpdate();
-        return balance;
    }
```
## Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.0",
settings: {
  optimizer: {
    enabled: true,
    runs: 1000,
  },
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.