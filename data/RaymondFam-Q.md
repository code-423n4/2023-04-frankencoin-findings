## Inadequate calculateFreedAmount() and calculateAssignedReserve()
In `Frankencoin.calculateFreedAmount()`, users will have to separately figure out on their own what exact `amountExcludingReserve` is needed in order to fully repay their loans via [`burnWithReserve()`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L251-L257) when [`currentReserve < minterReserve_`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L238). Consider refactoring `calculateFreedAmount()` that will take in another parameter, i.e. the intended [`freedAmount`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L252) and correspondingly returns an additional variable, i.e, the required `_amountExcludingReserve`.

The same refactor consideration should also be given to `Frankencoin.calculateAssignedReserve()` so owner will know what additional `ZCHF`, i.e the loss, she will need to have in her wallet prior to calling [`burnFrom()`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L223-L229).  

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
## Position owner could shill bid after increasing the price
A position owner attempting to mint more `ZCHF` by [increasing the price](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L159-L167) without increasing the collateral  could technically shill bid any challenge launched against it. This will ensure the owner take back his collateral above market price at the worst and per chance the owner could profit from a higher bid than what she has shilled or even have the challenged averted by the next bidders. 

## Hardcode 7 days
In MintingHub.sol, [`_initPeriodSeconds`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L99), i.e. `initPeriod` has been harcoded to [7 days](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L64). It might seem appropriate or optimized for now but could be too short or too long when market changed and/or more minters surfaced, making this specific input parameter no longer competitive or popular.

Consider soft coding it where possible. 

## Incorrect parameter on isPosition()
`isPosition(spender)` generally returns zero address since it is unlikely that the minter (spender) registers itself as a position. It is non-critical here since the first condition should always suffice. Nevertheless, consider having the affected code refactored as follows unless the protocol has an intended plan other than the one aforementioned:

[File: Frankencoin.sol#L102-L111](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L102-L111)

```diff
   function allowanceInternal(address owner, address spender) internal view override returns (uint256) {
      uint256 explicit = super.allowanceInternal(owner, spender);
      if (explicit > 0){
         return explicit; // don't waste gas checking minter
-      } else if (isMinter(spender) || isMinter(isPosition(spender))){
+      } else if (isMinter(spender) || isMinter(isPosition(owner))){
         return INFINITY;
      } else {
         return 0;
      }
   }
```
## Sanity checks at the constructor
Adequate zero address, zero value and boundary checks should be implemented at the constructor particularly in [Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L50-L70) considering a single casual mistake could end up costing 1000 `ZCHF` of [`OPENING_FEE`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L20). Misentering a 0 `limit`, having `mintingFeePPM` and  `reserveContribution` 1e18 instead of PPM scaled etc are just some of the costly mistakes that could transpire particularly when the user is interacting from a non-UI contract. 

## Frankencoin contract owner possesses too sensitive privileges
The deployer of the contract has too many privileges relative to standard users. The consequence is disastrous if the contract owner's private key has been compromised. 

For a Frankencoin Project project this grand, it increases the likelihood that the owner will be targeted by an attacker, especially given the insufficient protection on sensitive owner private keys. The concentration of privileges creates a single point of failure, specifically on calling `mint()` to mess up with `totalSupply()` on the healthy circulation of `ZCHF`, granting minter role to any party at his/her discretion etc. 

Consider:
1. splitting privileges (e.g. via the multisig option) to ensure that no one address has excessive ownership of the system,
2. clearly documenting the functions and implementations the owner can change,
3. documenting the risks associated with privileged users and single points of failure, and
4. ensuring that users are aware of all the risks associated with the system.

## Comment and code mismatch
In Position.sol, the [NatSpec comment](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L116-L119) of `getUsableMint()` says that the non-usable mint is used to buy reserve pool shares, whereas `zchf.mint(target, amount, reserveContribution, calculateCurrentFee())` does not have evidence alleging that FPS shares are being bought:

[File: Frankencoin.sol#L165-L170](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L165-L170)

```solidity
   function mint(address _target, uint256 _amount, uint32 _reservePPM, uint32 _feesPPM) override external minterOnly {
      uint256 usableMint = (_amount * (1000_000 - _feesPPM - _reservePPM)) / 1000_000; // rounding down is fine
      _mint(_target, usableMint);
      _mint(address(reserve), _amount - usableMint); // rest goes to equity as reserves or as fees
      minterReserveE6 += _amount * _reservePPM; // minter reserve must be kept accurately in order to ensure we can get back to exactly 0
   }
```
As can be seen from the code block above, rest goes to equity as reserves or as fees only. It is not being routed through [`transferAndCall()`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L162-L168) to invoke [`IERC677Receiver(recipient).onTokenTransfer(msg.sender, amount, data)`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L241-L255) for [minting](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L248) of `FPS`. Neither is `redeem()` called to [burn](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L278) any FPS when [`calculateAssignedReserve()`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L204-L213) or [`freedAmount - _amountExcludingReserve`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L254) is transferred back to the position owner.  

## Correct placement of chf.transferFrom()
Consider moving the critical `chf.transferFrom()` from `mint()` to `mintInternal()` just like it has been done so in `burnInternal()` just in case it can be bricked dodging the need to send in `CHF` when minting new `ZCHF`:

[File: StablecoinBridge.sol#L44-L53](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L44-L53)

```diff
    function mint(address target, uint256 amount) public {
-        chf.transferFrom(msg.sender, address(this), amount);
        mintInternal(target, amount);
    }

    function mintInternal(address target, uint256 amount) internal {
        require(block.timestamp <= horizon, "expired");
        require(chf.balanceOf(address(this)) <= limit, "limit");
+        chf.transferFrom(msg.sender, address(this), amount);
        zchf.mint(target, amount);
    }
``` 
## System pauser
Consider implementing a system pauser on critical contracts particularly when it relates to Frankencoin.sol just in case it has been seriously compromised as far as the minting capabilities are of primary concern. 