# QA Report

## Findings Summary

| ID | Issue | Severity |
| - | - | - |
| [L-01] | Users have no way of revoking their signatures | Low |
| [L-02] | `MIN_APPLICATION_PERIOD` should always be more than 0 | Low |
| [L-03] | Dangerous assumption of stablecoin peg | Low |
| [L-04] | Incorrect code in `restructureCapTable()` | Low |
| [L-05] | Minor inconsistency with comment in `calculateProceeds()` | Low |
| [L-06] | Users can lose a significant amount of votes by transferring shares to themselves | Low |
| [L-07] | One share will always be stuck in the `Equity` contract | Low |
| [L-08] | Misleading implementation of vote delegation | Low |
| [N-01] | `_transfer()` doesn't check that `sender != address(0)` | Non-Critical |
| [N-02] | Correctness of `_cubicRoot()` | Non-Critical |

## [L-01] Users have no way of revoking their signatures

In [`ERC20PermitLight.sol`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol), users can use signatures to approve other users as spenders. Spenders will then use the `permit()` function to verify the signature and gain allowance.

Currently, users have no way of revoking their signatures once it is signed. This could result in a misuse of approvals:
- Alice provides a signature to approve Bob to spend 100 tokens.
- Before `deadline` is passed, Bob's account is hacked.
- As Alice has no way of revoking the signature, Bob's account uses the signature to approve and spend Alice's tokens.

### Recommendation

Implement a function for users to revoke their own signatures, such as a function that increments the nonce of `msg.sender` when called:

```solidity
function incrementNonce() public {
    nonces[msg.sender]++;
}
```

## [L-02] `MIN_APPLICATION_PERIOD` should always be more than 0

In [`Frankencoin.sol`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol), if `MIN_APPLICATION_PERIOD` is ever set to 0, anyone can call [`suggestMinter()`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L72-L90) with `_applicationPeriod = 0`, instantly becoming a minter. They will then be able to call sensitive minter functions, such as [`mint()`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L161-L174).

### Recommendation

Ensure that `MIN_APPLICATION_PERIOD` is never set to 0. This can be done in the constructor:

[Frankencoin.sol#L59-L62](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L59-L62)

```diff
    constructor(uint256 _minApplicationPeriod) ERC20(18){
+      require(_minApplicationPeriod != 0, "MIN_APPLICATION_PERIOD cannot be 0");
       MIN_APPLICATION_PERIOD = _minApplicationPeriod;
       reserve = new Equity(this);
    }
```

## [L-03] Dangerous assumption of stablecoin peg

In [`StablecoinBridge.sol`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol), users can deposit a chosen stablecoin in exchange for Frankencoin:

[StablecoinBridge.sol#L40-L53](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L40-L53)

```solidity
/**
* Mint the target amount of Frankencoins, taking the equal amount of source coins from the sender.
* This only works if an allowance for the source coins has been set and the caller has enough of them.
*/
function mint(address target, uint256 amount) public {
    chf.transferFrom(msg.sender, address(this), amount);
    mintInternal(target, amount);
}

function mintInternal(address target, uint256 amount) internal {
    require(block.timestamp <= horizon, "expired");
    require(chf.balanceOf(address(this)) <= limit, "limit");
    zchf.mint(target, amount);
}
```

Where:
- `chf` - Address of the input stablecoin contract.
- `zchf` - Address of Frankencoin contract. 

This implementation assumes that the input stablecoin will always have a 1:1 value with Frankencoin. However, in the unlikely situation that the input stablecoin depegs, users can use this stablecoin bridge to mint Frankencoin at a discount, thereby harming the protocol.

### Recommendation

Use a price oracle to ensure the input stablecoin price is acceptable. Alternatively, implement some method for a trusted user to intervene, such as allowing a trusted user to pause minting in the event the input stablecoin depegs.

## [L-04] Incorrect code in `restructureCapTable()`

In [`Equity.sol`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol), the `restructureCapTable()` function contains an error:

[Equity.sol#L309-L316](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L309-L316)

```solidity
function restructureCapTable(address[] calldata helpers, address[] calldata addressesToWipe) public {
    require(zchf.equity() < MINIMUM_EQUITY);
    checkQualified(msg.sender, helpers);
    for (uint256 i = 0; i<addressesToWipe.length; i++){
        address current = addressesToWipe[0]; // @audit Should be addressesToWipe[i]
        _burn(current, balanceOf(current));
    }
}
```

## [L-05] Minor inconsistency with comment in `calculateProceeds()`

In [`Equity.sol`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol), the `calculateProceeds()` function is as shown:

[Equity.sol#L290-L297](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L290-L297)

```solidity
function calculateProceeds(uint256 shares) public view returns (uint256) {
    uint256 totalShares = totalSupply();
    uint256 capital = zchf.equity();
    require(shares + ONE_DEC18 < totalShares, "too many shares"); // make sure there is always at least one share
    uint256 newTotalShares = totalShares - shares;
    uint256 newCapital = _mulD18(capital, _power3(_divD18(newTotalShares, totalShares)));
    return capital - newCapital;
}
```

The comment states that a minimum of one share (`ONE_DEC18`) must remain in the contract. However, the require statement actually ensures that `totalShares` is never below `ONE_DEC18 + 1`. 

### Recommendation

Change the require statement to the following:

```solidity
require(totalShares - shares >= ONE_DEC18, "too many shares"); // make sure there is always at least one share
```

## [L-06] Users can lose a significant amount of votes by transferring shares to themselves 

In [`Equity.sol`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol), `adjustRecipientVoteAnchor()` is called by the `_beforeTokenTransfer()` hook to adjust a recevier's `voteAnchor`:

[Equity.sol#L150-L167](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L150-L167)

```solidity
/**
* @notice the vote anchor of the recipient is moved forward such that the number of calculated
* votes does not change despite the higher balance.
* @param to        receiver address
* @param amount    amount to be received
* @return the number of votes lost due to rounding errors
*/
function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){
    if (to != address(0x0)) {
        uint256 recipientVotes = votes(to); // for example 21 if 7 shares were held for 3 blocks
        uint256 newbalance = balanceOf(to) + amount; // for example 11 if 4 shares are added
        voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); // new example anchor is only 21 / 11 = 1 block in the past
        return recipientVotes % newbalance; // we have lost 21 % 11 = 10 votes
    } else {
        // optimization for burn, vote anchor of null address does not matter
        return 0;
    }
}
```

The function is used to ensure a user has the same amount of votes before and after a transfer of shares. 

However, if a user transfers all his shares to himself, `newBalance` will be two times of his actual balance, causing his `voteAnchor` to become larger than intended. Therefore, he will lose a significant amount of votes.

### Recommendation

Make the following change to the `_beforeTokenTransfer()` hook:

[Equity.sol#L112-L114](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L112-L114)

```diff
     function _beforeTokenTransfer(address from, address to, uint256 amount) override internal {
         super._beforeTokenTransfer(from, to, amount);
-        if (amount > 0){
+        if (amount > 0 && from != to){
```

This ensures users do not lose votes in the unlikely event where they transfer shares to themselves.

## [L-07] One share will always be stuck in the `Equity` contract

In `Equity.sol`, the following require statement ensures that the last share can never be redeemed from the contract:

[Equity.sol#L293](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L293)

```solidity
require(shares + ONE_DEC18 < totalShares, "too many shares"); // make sure there is always at least one share
```

In the unlikely event where everyone wants to redeem their shares, the last redeemer will never be able to redeem his last share. Therefore, he will always have some Frankencoin stuck in the contract.

## [L-08] Misleading implementation of vote delegation

In [`Equity.sol`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol), users can delegate their votes to a delegatee using the [`delegateVoteTo()`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L216-L223) function.

The `canVoteFor()` function is then used to check if `delegate` is a delegatee of `owner`:

[Equity.sol#L225-L233](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L225-L233)

```solidity
function canVoteFor(address delegate, address owner) internal view returns (bool) {
    if (owner == delegate){
        return true;
    } else if (owner == address(0x0)){
        return false;
    } else {
        return canVoteFor(delegate, delegates[owner]);
    }
}
```

However, due to its recursive implementation, delegations can be chained to combined the votes of multiple users, similar to a linked list. For example:

- Alice's delegatee is Bob.
- Bob's delegatee is Charlie.

Due to the chained delegation, Charlie's voting power is increased by the votes of both Alice and Bob.

However, this becomes an issue if Alice wishes to delegate votes to **only** Bob, and no one else. Once she sets Bob as her delegatee, Bob's delegatee will also gain voting power from her votes; Alice has no control over this.

### Recommendation

Consider removing this chained delegation functionality in `canVoteFor()`:

```solidity
function canVoteFor(address delegate, address owner) internal view returns (bool) {
    return owner == delegate;
}
```

## [N-01] `_transfer()` doesn't check that `sender != address(0)`

In [`ERC20.sol`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol), the `_transfer()` function does not ensure that the `sender` is not  `address(0)`:

[ERC20.sol#L151-L152](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L151-L152)

```solidity
function _transfer(address sender, address recipient, uint256 amount) internal virtual {
    require(recipient != address(0));
```

This differs from [OpenZeppelin's ERC20 implementation](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol#L222-L224).

### Recommendation 

Add the following check to `_transfer()`:

[ERC20.sol#L151-L152](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L151-L152)

```diff
     function _transfer(address sender, address recipient, uint256 amount) internal virtual {
+        require(sender != address(0));
         require(recipient != address(0));
```

## [N-02] Correctness of `_cubicRoot()`

The following Foundry fuzz test can be used to verify the correctness of the [`_cubicRoot()`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L12-L29) function:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../contracts/MathUtil.sol";

contract MathUtilTest is Test, MathUtil {
    function testFuzz_cubicRoot(uint256 v) public {
        v = bound(v, ONE_DEC18, 38597363079105398489064286242003304715461606497555392182630);
        uint256 result = _power3(_cubicRoot(v));
        if (result != v) {
            uint256 err = v > result ? v - result : result - v;
            assertLt(err * 1e8, v);
        } 
    }
}
```

The test proves that `_cubicRoot()` has a margin of error below 0.000001% when `_v >= `1e18`.

Additionally, the function has the following limitations:
- `_cubicRoot()` reverts if `_v` is greater than `38597363079105398489064286242003304715461606497555392182630`.
- `_cubicRoot()` is incorrect for extremely small values of `_v`. For example:
  - `_cubicRoot(0)` returns `7812500000000000`.
  - `_cubicRoot(1)` returns `7812500000003509`.

Nevertheless, these limitations do not pose any issue in the current implementation of the protocol.