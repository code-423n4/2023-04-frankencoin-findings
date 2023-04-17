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

[L-02] Ownership may be burned
================================
In the contract `Ownable.sol` it was observed that the function `transferOwnership` does not check transfer to `address(0)` which effectively burn the ownership.

## Proof Of Concept
* https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L27-L43
```solidity
    /**
     * @dev Transfers ownership of the contract to a new account (`newOwner`).
     * Can only be called by the current owner.
     */
    function transferOwnership(address newOwner) public onlyOwner {
        setOwner(newOwner);
    }

    /**
     * @dev Transfers ownership of the contract to a new account (`newOwner`).
     * Internal function without access restriction.
     */
    function setOwner(address newOwner) internal {
        address oldOwner = owner;
        owner = newOwner;
        emit OwnershipTransferred(oldOwner, newOwner);
    }

```
The function `transferOwnership` above takes the address of `newOwner` as an argument and is directly passed to the function `setOwner`. However, there is no checks in both of these functions to ensure that the `newOwner` is not `address(0)`. This means that the owner can transfer the ownership to `address(0)` which effectively burn the ownership.

## Recommended mitigation steps
It is recommended to implement a validation check to ensure that the ownership is not transferred to `address(0)`.
Example:
```solidity

function transferOwnership(address newOwner) public onlyOwner {
    require(newOwner != address(0), "New owner cannot be zero address");
    setOwner(newOwner);
}


```

[L-03] Potential name confusion in the function `votes` in the contracts `Equity.sol`
=====================================================================================
The issue with the `votes` function is that it has two functions with the same name, which causes a name conflict. The first `votes` function takes only one argument, which is the sender's address and returns the number of votes for the sender. The second `votes` function takes two arguments, one sender's address, and an array of helper addresses, and returns the total number of votes for both the sender and the helper addresses.

This is problematic because the two functions have the same name, but different input parameters, which can lead to confusion and potential errors when calling the function. When calling the function, it may not be clear which version of the votes function is being called, especially if the arguments are not explicitly stated. This could result in unexpected behavior and may cause errors in the contract.

## Proof Of Concept
* https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L190-#L202

## Recommended mitigation steps
To resolve this issue, one of the `votes` functions should be renamed to avoid the naming conflict. For example, the first function could be renamed `votesForSender` to clarify its purpose, while the second function could remain as `votes` since it takes both the sender and the helper addresses as arguments. By doing this, the contract code becomes more explicit and easy to understand.
