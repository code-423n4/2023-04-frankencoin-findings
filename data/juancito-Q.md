# QA Report

## Low Issues

• [LOW-1] Frontrunning suggestMinter may lead to stolen funds

• [LOW-2] Not validating MIN_APPLICATION_PERIOD can lead to stolen funds

• [LOW-3] MIN_HOLDING_DURATION will not hold a correct value if deployed on other network

• [LOW-4] `restructureCapTable()` only wipes out the first address on the list

• [LOW-5] Positions should be expired when `block.timestamp = expiration`

• [LOW-6] No pause mechanism in case of depeg of XCHF token

• [LOW-7] Tokens with very large decimals will not work as expected

• [LOW-8] minBid can be bypassed to bid indefinitely for small amounts

• [LOW-9] No way to transfer minter role or rennounce to it

• [LOW-10] ERC-777 tokens can lead to re-entrancy vulnerabilities

• [LOW-11] Challenges can be split after they end

### [LOW-1] Frontrunning suggestMinter may lead to stolen funds

`Frankencoin::suggestMinter` does not validate `_applicationPeriod` and `_applicationFee` when `totalSupply()` is `0`

This can be useful for the admins to create the first minter.

Nevertheless the function can be called by anyone until some tokens are minted.

#### Impact

In the worst scenario the admins can deploy all contracts. Send funds to the `StablecoinBridge`, and then decide to suggest the first minter and mint some coins.

An attacker can call `suggestMinter` as soon as the contracts are created and some funds are sent to burn them and retrieve the paired stable coin provided by the bridge.

On a less dramatic scenario anyone can frontrun the `suggestMinter` function, which will create a new minter for the attacker. This will result in the contracts having to be re-deployed.

#### Proof of Concept

`Frankencoin::suggestMinter` does not validate `_applicationPeriod` and `_applicationFee` when `totalSupply()` is `0`:

```solidity
      if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
```

[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84-L85)

#### Recommended Mitigation Steps

Add some admin permission to assign the first minter, or deploy everything through a deployer contract, and call `suggestMinter` in it, so that it cannot be front-runned.

### [LOW-2] Not validating MIN_APPLICATION_PERIOD can lead to stolen funds

`MIN_APPLICATION_PERIOD` is defined on the `Frankencoin` constructor and is immutable. In case the contracts are deployed with `_minApplicationPeriod = 0`, an attacker can become a minter, burn the token and steal assets on other contracts like the `StablecoinBridge`.

```solidity
   constructor(uint256 _minApplicationPeriod) ERC20(18){
      MIN_APPLICATION_PERIOD = _minApplicationPeriod;
      reserve = new Equity(this);
   }
```

[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L59-L62)

#### Recommended Mitigation Steps

Validate that the `MIN_APPLICATION_PERIOD` is greater than some minimum value in the constructor.

### [LOW-3] MIN_HOLDING_DURATION will not hold a correct value if deployed on other network

`MIN_HOLDING_DURATION` in `Equity` is calculated assuming that it will be only deployed in Ethereum Mainnet, where each block is added every 12 seconds.

```solidity
uint256 public constant MIN_HOLDING_DURATION = 90*7200 << BLOCK_TIME_RESOLUTION_BITS; // Set to 5 for local testing
```

[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L59)

That will not be true if the contract is deployed on other networks

#### Recommended Mitigation Steps

Define `MIN_HOLDING_DURATION` via a variable in the constructor, or add a comment on the code make clear this issue.

### [LOW-4] `restructureCapTable()` only wipes out the first address on the list

There is an error on the line `address current = addressesToWipe[0];`. It is always wiping out `addressesToWipe[0]`, instead of `addressesToWipe[i]`. 

If not double-checked, this could lead to not wiping out the expected users. It is still possible to mitigate it by wiping out users one by one.

```solidity
function restructureCapTable(address[] calldata helpers, address[] calldata addressesToWipe) public {
    require(zchf.equity() < MINIMUM_EQUITY);
    checkQualified(msg.sender, helpers);
    for (uint256 i = 0; i<addressesToWipe.length; i++){
        address current = addressesToWipe[0]; // @audit
        _burn(current, balanceOf(current));
    }
}
```

#### Recommended Mitigation Steps

Pass the iteration variable `i`:

```diff
-   address current = addressesToWipe[0];
+   address current = addressesToWipe[i];
```

### [LOW-5] Positions should be expired when `block.timestamp = expiration`

The `alive` modifier in `Position` does not revert when `block.timestamp == expiration`:

```solidity
    block.timestamp > expiration) revert Expired();
```

[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L367)

This by itself only allows positions to operate on that exact block, but if the `noCooldown` modifier had the same issue, it could lead to exploits when all values are equal.

```solidity
    if (block.timestamp <= cooldown) revert Hot();
```

It was noticed that some changes to `>` operatos have been performed when replacing `require` statements with `revert` statements.

It is important to check this subtle differences to prevent any issue.

On top of that, the `tryAvertChallenge` already checks `block.timestamp >= expiration` for expiration:

```solidity
    /**
     * @notice check whether challenge can be averted
     * @param _collateralAmount   amount of collateral challenged (dec18)
     * @param _bidAmountZCHF      bid amount in ZCHF (dec18)
     * @return true if challenge can be averted
     */
    function tryAvertChallenge(uint256 _collateralAmount, uint256 _bidAmountZCHF) external onlyHub returns (bool) {
        if (block.timestamp >= expiration){
```

[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L299-L305)

#### Recommended Mitigation Steps

For code consistency, and to prevent any possible issues with the exact block of expiration:

```solidity
-    block.timestamp > expiration) revert Expired();
+    block.timestamp >= expiration) revert Expired();
```

### [LOW-6] No pause mechanism in case of depeg of XCHF token

The system allows minting 1-1 FrankenCoin with the same amount of XCHF tokens. In the case the stable coin XCHF depegs from its value it will greatly affect the value of the FrankenCoin ZCHF token, as they can be redeemed immediately.

#### Recommended Mitigation Steps

It should be up for consideration the implementation of a pause function on the StablecoinBridge to prevent minting tokens 1-1 in case of emergency.

### [LOW-7] Tokens with very large decimals will not work as expected

Although not common, it is possible that ERC-20 tokens have `decimals() > 36`. The current system acknowledges it, but does not prevent anyone from creating a position with them. This will result in the token not working as expected.

```solidity
     * @param _liqPrice          Liquidation price with (36 - token decimals) decimals,
     *                           e.g. 18 decimals for an 18 decimal collateral, 36 decimals for a 0 decimal collateral.
```

[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L83-L84)

#### Recommended Mitigation Steps

Validate that tokens have < than 36 decimals or the desired value

### [LOW-8] minBid can be bypassed to bid indefinitely for small amounts

`MintingHub::minBid()` returns the same number as the `challenge.bid` for small numbers:

```solidity
function minBid(Challenge storage challenge) internal view returns (uint256) {
    return (challenge.bid * 1005) / 1000;
}
```

For `challenge.bid <= 199`, the result of `minBid` is the same as the bid because of the precision loss error.

`minBid` is used in the `bid` function to check if the bid should be postponed:

```solidity
    if (_bidAmountZCHF < minBid(challenge)) revert BidTooLow(_bidAmountZCHF, minBid(challenge)); // @audit
    uint256 earliestEnd = block.timestamp + 30 minutes;
    if (earliestEnd >= challenge.end) {
        // bump remaining time like ebay does when last minute bids come in
        // An attacker trying to postpone the challenge forever must increase the bid by 0.5%
        // every 30 minutes, or double it every three days, making the attack hard to sustain
        // for a prolonged period of time.
        challenge.end = earliestEnd;
    }
```

#### Recommended Mitigation Steps

Replace the `<` with `<=`. That way it will revert when the amounts are low enough to return the same number.

```solidity
-    if (_bidAmountZCHF < minBid(challenge)) revert BidTooLow(_bidAmountZCHF, minBid(challenge));
+    if (_bidAmountZCHF <= minBid(challenge)) revert BidTooLow(_bidAmountZCHF, minBid(challenge));
```

### [LOW-9] No way to transfer minter role or rennounce to it

Minters have a lot of power, and put the protocol at risk if their accounts could get compromised. But they have no way to opt-out if they think their opsec is not adequate at some point. This could also be useful in case there is some multi-sig minter for example.

#### Recommended Mitigation Steps

Allow minters to rennounce to their roles.

Allow minters to "transfer" their minter role to another address.

Implementation should also take care of positions that were registered with those minters via `registerPosition()`.

### [LOW-10] ERC-777 tokens can lead to re-entrancy vulnerabilities

ERC-777 behave like ERC-20 tokens, but they make a callback when tokens are transfered.

#### Impact

Positions containing ERC-777 tokens as collateral may be victim of re-entrancy attacks.

The possible impacts are unrestricted minting of ZCHF tokens via `end`, and stealing ZCHF tokens from the `MintingHub`, both critical.

On top of that, some inconsistency on the positions' storage can be result of these unexpected behavior.

The actual impact relies on the `minters` of the protocol allowing the possibility of using ERC-777 tokens as collateral or denying them.

#### Proof of Concept

These functions in the `MintingHub` are suceptible to re-entrancy attacks. An attacker can perform them by first launching a challenge, and then calling the respected functions. As the bidder and challenger will be the same, the collateral will be transfered between attacker accounts.

`end()` calls `returnCollateral` early on the function, before the `challenge` is deleted. So, it can be re-entered to mint extra tokens via `zchf.notifyLoss`:

```solidity
    // returnCollateral()
    challenge.position.collateral().transfer(msg.sender, challenge.size);
```

[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L211)

`bid()` can be re-entered when the bid is high enough to avert the challenge.

First, the attacker needs to have a previous bid, or the attacker can create one.

The attacker would then be able to steal the initial bid multiple times via the `zchf.transfer(challenge.bidder, challenge.bid);`. Assets will be taken from the `MintingHub` contract.

The re-entrancy can be executed by calling the function with a value big enough to avert the challenge:

```solidity
    // bid()
    challenge.position.collateral().transfer(challenge.challenger, challenge.size); // return the challenger's collateral
```

[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L294)

#### Recommended Mitigation Steps

Add re-entrancy guards to functions that transfer collateral, and implement the Checks-Effects-Interaction pattern. Or disallow the use of ERC-777 tokens as collateral.

[Link to code](https://github.com/Frankencoin-ZCHF/FrankenCoin/blob/main/contracts/MintingHub.sol#L88-L113)

### [LOW-11] Challenges can be split after they end

#### Impact

Griefing users by diving their already ended challenges

#### Proof of Concept

```solidity
function testBidAfterEnd() public {
    Position position = Position(initPosition());

    skip(7 * 86_400 + 60);

    User challenger = new User(zchf);
    col.mint(address(challenger), 1001);
    uint256 challengeNumber = challenger.challenge(hub, address(position), 1001);

    uint256 bidAmount = 1 ether;
    bob.obtainFrankencoins(swap, 1 ether);
    bob.bid(hub, challengeNumber, bidAmount);

    skip(7 * 86_400 + 60);

    vm.startPrank(address(alice));
    hub.splitChallenge(challengeNumber, 500); // @audit
    hub.end(challengeNumber); // @audit
    vm.stopPrank();
}
```

#### Recommended Mitigation Steps

Add `if (block.timestamp >= challenge.end) revert TooLate();` to the `splitChallenge` function

## Non Critical Issues

• [NC-1] No way to track challenges created by a user

• [NC-2] Equity tokens sent to oneself are processed to update votes

• [NC-3] Misleading comment about position `start` value

• [NC-4] Rebasing tokens can lead to bad accountability of the positions

### [NC-1] No way to track challenges created by a user

Challenges launched via the MintingHub are added to a general `challenges` array.

But it will be troublesome to track all challenges created by a user as long as that array grows, especially considering that anyone can split challenges and make that array grow faster than expected.

#### Recommended Mitigation Steps

Create a mapping that tracks the challenges launched by users, and also adds them when challenges are splitted.

### [NC-2] Equity tokens sent to oneself are processed to update votes

Tokens sent are pre-processed in `_beforeTokenTransfer`. The `adjustRecipientVoteAnchor` and `adjustTotalVotes` are called, where calculations for the user and the total votes are made.

In the current codebase it doesn't generate any issues, but any uncatched precision loss or some subtle changes to those calculations could be used to perform some exploit.

```solidity
    function _beforeTokenTransfer(address from, address to, uint256 amount) override internal {
        super._beforeTokenTransfer(from, to, amount);
        if (amount > 0){
            // No need to adjust the sender votes. When they send out 10% of their shares, they also lose 10% of
            // their votes so everything falls nicely into place.
            // Recipient votes should stay the same, but grow faster in the future, requiring an adjustment of the anchor.
            uint256 roundingLoss = adjustRecipientVoteAnchor(to, amount);
            // The total also must be adjusted and kept accurate by taking into account the rounding error.
            adjustTotalVotes(from, amount, roundingLoss);
        }
    }
```

[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L112-L122)

#### Recommended Mitigation Steps

The current implementation makes the users sending the tokens lose the votes. This can be the expected behavior.

The following suggestions adds some safety to the function, but in exchange, it changes the original functionality. With this change, users will not lose votes if they send tokens to themselves:

```diff
    function _beforeTokenTransfer(address from, address to, uint256 amount) override internal {
        super._beforeTokenTransfer(from, to, amount);
-        if (amount > 0){
+        if (amount > 0 && from != to){
            // No need to adjust the sender votes. When they send out 10% of their shares, they also lose 10% of
            // their votes so everything falls nicely into place.
            // Recipient votes should stay the same, but grow faster in the future, requiring an adjustment of the anchor.
            uint256 roundingLoss = adjustRecipientVoteAnchor(to, amount);
            // The total also must be adjusted and kept accurate by taking into account the rounding error.
            adjustTotalVotes(from, amount, roundingLoss);
        }
    }
```

### [NC-3] Misleading comment about position `start` value

The code suggests that there is "one week" time to deny the position

```solidity
start = block.timestamp + initPeriod; // one week time to deny the position
```

[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L64)

But some lines before it specifies `3 days`:

```solidity
require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values
```

[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L53)

#### Recommended Mitigation Steps

Fix the comment or the required value

### [NC-4] Rebasing tokens can lead to bad accountability of the positions

Rebasing tokens change the `balanceOf` value of the accounts that hold their tokens.

Using rebasing tokens as collateral for positions can lead to positions minting more tokens than expected, or challenges created, averted or won with different amounts than expected.

I would suggest not allowing rebasing tokens to be used on the protocol.