## Inadequate calculateFreedAmount()
Users will have to separately figure out on their own what exact `amountExcludingReserve` is needed in order to fully repay their loans when [`currentReserve < minterReserve_`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L238). Consider refactoring `calculateFreedAmount()` that will take in another parameter, i.e. the intended [`freedAmount`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L252) and correspondingly returns an additional variable, i.e, the required `_amountExcludingReserve`.

## Chain reorganization attack
As denoted in the [Moralis academy article](https://academy.moralis.io/blog/what-is-chain-reorganization):

"... If a node receives a new chain that’s longer than its current active chain of blocks, it will do a chain reorg to adopt the new chain, regardless of how long it is."

Depending on the outcome, if it ended up placing the transaction earlier than anticipated, many of the system function calls could backfire. For instance, a bidder bidding on a trimmed position could run into DoS because of size mismatch. Similarly, a postion owner attempting to mint more `ZCHF` could also run into DoS due to `cooldown` not over yet etc.

(Note: On Ethereum this is unlikely but this is meant for contracts going to be deployed on any compatible EVM chain many of which like Polygon, Optimism, Arbitrum are frequently reorganized.) 

## Typo mistakes
[File: Position.sol#L358](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L358)

```diff
-     * This is also a good creterion when deciding whether it should be shown in a frontend.
+     * This is also a good criterion when deciding whether it should be shown in a frontend.
```
[File: Equity.sol#L34](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L34)

```diff
-     * I.e., the supply is proporational to the cubic root of the reserve and the price is proportional to the
+     * I.e., the supply is proportional to the cubic root of the reserve and the price is proportional to the
```
[File: Equity.sol#L73](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L73)

```diff
-     * cap of 3,000,000,000,000,000,000 CHF. This limit could in theory be reached in times of hyper inflaction.
+     * cap of 3,000,000,000,000,000,000 CHF. This limit could in theory be reached in times of hyper inflation.
```
[File: Equity.sol#L207](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L207)

```diff
-     * helpes are necessary and to identify them by scanning the blockchain for Delegation events.
+     * helpers are necessary and to identify them by scanning the blockchain for Delegation events.
```
## Include descriptive variables in mappings
Where possible, implement mappings with descriptive variables included for better readability.

Here are some of the instances entailed:

[File: MintingHub.sol#L37](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L37)

```solidity
    mapping (address /** col */ => mapping (address => uint256)) public pendingReturns;
```
[File: Frankencoin.sol#L45-L50](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L45-L50)

```solidity
   mapping (address => uint256) public minters;

   /**
    * List of positions that are allowed to mint and the minter that registered them.
    */
   mapping (address => address) public positions;
```
## Non-compliant contract layout with Solidity's Style Guide
According to Solidity's Style Guide below:

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

In order to help readers identify which functions they can call, and find the constructor and fallback definitions more easily, functions should be grouped according to their visibility and ordered in the following manner:

constructor, receive function (if exists), fallback function (if exists), external, public, internal, private

And, within a grouping, place the view and pure functions last.

Additionally, inside each contract, library or interface, use the following order:

type declarations, state variables, events, modifiers, functions

Consider adhering to the above guidelines for all contract instances entailed.

## Tokens accidentally sent to the contract cannot be recovered
It is deemed unrecoverable if any of the ERC20 tokens accidentally arrive at the contract addresses, which has happened to many popular projects. Consider adding a recovery code to your critical contracts just like it has been implemented in contract Position:

[File: Position.sol#L245-L255](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L245-L255)

```solidity
    /**
     * Withdraw any ERC20 token that might have ended up on this address.
     * Withdrawing collateral is subject to the same restrictions as withdrawCollateral(...).
     */
    function withdraw(address token, address target, uint256 amount) external onlyOwner {
        if (token == address(collateral)){
            withdrawCollateral(target, amount);
        } else {
            IERC20(token).transfer(target, amount);
        }
    }
```
## Parameterized custom errors
Custom errors can be parameterized to better reflect their respective error messages if need be.

For instance, the custom error instance below may be refactored as follows:

[File: Position.sol#L103](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L103)

```diff
-    error TooLate();
+    error TooLate(uint256 _start);
```
[File: Position.sol#L110](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L110)

```diff
-        if (block.timestamp >= start) revert TooLate();
+        if (block.timestamp >= start) revert TooLate(start);
```