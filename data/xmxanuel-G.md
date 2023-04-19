# Gas Report

### 1. Helpers unique check can be performed more gas efficient in O(n) for Equity.votes

Currently, a second loop is required to check if no duplicates exist. Since this would count voting power twice.

```solidity=
    function votes(address sender, address[] calldata helpers) public view returns (uint256) {
        uint256 _votes = votes(sender);
        for (uint i=0; i<helpers.length; i++){
            address current = helpers[i];
            require(current != sender);
            require(canVoteFor(sender, current));
            for (uint j=i+1; j<helpers.length; j++){
                require(current != helpers[j]); // ensure helper unique
            }
            _votes += votes(current);
        }
        return _votes;
    }
```

However, this check can be performed in `O(n)` without additional memory or storage. The contract only needs to require the `helpers` address array to be sorted. This would easily allow to identify duplicates.

```solidity
    function checkDuplicatesAndSorted(address[] calldata helpers) external pure returns (bool ok) {
        if (helpers.length <= 1) {
            return true;
        }
        address prevAddress = helpers[0];
        for (uint i = 1; i < helpers.length; i++) {
            if (helpers[i] <= prevAddress) {
                return false;
            }
            prevAddress = helpers[i];
        }
        return true;
    }
```


### 2. Make MintingHub.bid transaction reverts more gas efficient
In case the `bid` transaction reverts the gas still needs to be paid by the user.

It would be more gas efficient to check the `require` as first step in the bid function.

Together, with the the `transferFrom` of `ZHF` tokens.

A forgotten approval might be the most common revert reason.

```diff=
    function bid(uint256 _challengeNumber, uint256 _bidAmountZCHF, uint256 expectedSize) external {
        Challenge storage challenge = challenges[_challengeNumber];
        if (block.timestamp >= challenge.end) revert TooLate();
        if (expectedSize != challenge.size) revert UnexpectedSize();
+       zchf.transferFrom(msg.sender, address(this), _bidAmountZCHF); 
+       if (_bidAmountZCHF < minBid(challenge)) revert BidTooLow(_bidAmountZCHF, minBid(challenge));
        if (challenge.bid > 0) {
            zchf.transfer(challenge.bidder, challenge.bid); // return old bid
        }
        emit NewBid(_challengeNumber, _bidAmountZCHF, msg.sender);
        // ask position if the bid was high enough to avert the challenge
        if (challenge.position.tryAvertChallenge(challenge.size, _bidAmountZCHF)) {
            // bid was high enough, let bidder buy collateral from challenger
            zchf.transferFrom(msg.sender, challenge.challenger, _bidAmountZCHF);
            challenge.position.collateral().transfer(msg.sender, challenge.size);
            emit ChallengeAverted(address(challenge.position), _challengeNumber);
            delete challenges[_challengeNumber];
        } else {
            // challenge is not averted, update bid
-            if (_bidAmountZCHF < minBid(challenge)) revert BidTooLow(_bidAmountZCHF, minBid(challenge));
            uint256 earliestEnd = block.timestamp + 30 minutes;
            if (earliestEnd >= challenge.end) {
                // bump remaining time like ebay does when last minute bids come in
                // An attacker trying to postpone the challenge forever must increase the bid by 0.5%
                // every 30 minutes, or double it every three days, making the attack hard to sustain
                // for a prolonged period of time.
                challenge.end = earliestEnd;
            }
-            zchf.transferFrom(msg.sender, address(this), _bidAmountZCHF);
            challenge.bid = _bidAmountZCHF;
            challenge.bidder = msg.sender;
        }
    }
```