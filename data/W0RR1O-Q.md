## QA Report (low/non-critical)

[L-01] Consider using OpenZeppelin’s `SafeCast` library to prevent unexpected overflows when casting from `uint256`
===================================================================================================================
In the contract `Equity.sol` two function `adjustTotalVotes` takes an argument `roundingLoss` of type `uint256` and a variable `lostVotes`of type `uint256` and downcast them to type `uint192`. Furthermore the function `adjustRecipientVoteAnchor` in the same contract sets two variables of type `uint256` which are `recipientVotes` and `newbalance` and downcast them to type `uint64` while performing some arithmetic to calculate the `voteAnchor`. However, since `totalVotesAtAnchor` should not exceed 68-bits and `voteAnchor` should not exceed 64-bits it is still considered as best practice to use OpenZepplin's `SafeCast` library to prevent any unexpected overflows from happening.

## Proof Of Concept
Function `adjustTotalVotes`:
* https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L144-#L149
Function `adjustRecipientVoteAnchor`:
* https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L157-#L167

## Recommended Mitigation Steps
Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256.
