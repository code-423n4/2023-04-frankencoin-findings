## Wrong comments at `Ownable` contract

https://github.com/Frankencoin-ZCHF/FrankenCoin/blob/main/contracts/Ownable.sol#L16-L17

In the `Ownable` contract, that is a modified copy from the Openzeppelin `Ownable` contract, the comments state that the owner will be by default the contract deployer. This statement is false because in the contract there's no constructor so the contract that inherit form this one must set the owner manually.

This issue has no real impact because the `Position` contract sets the owner in the constructor but this comment can lead to misunderstanding with other devs or with future versions of Frankencoin leading to a contract without owner.

## `checkQualified` won't revert when there are no shares minted or if it's called in the same block as the first shares are minted

https://github.com/Frankencoin-ZCHF/FrankenCoin/blob/main/contracts/Equity.sol#L209-L212

In the `Equity` contract, the `checkQualified` function should revert when the `sender` doesn't hold at least the 3% of the votes. This doesn't happen when there're not shares minted of when the function is called in the same block as the transaction that minted the first shares. 

This is because the function calls `totalVotes`, and this function makes the following calculation:
`totalVotesAtAnchor + totalSupply() * (anchorTime() - totalVotesAnchorTime)`
So, when there are no shares minted, `totalVotesAtAnchor` and `totalSupply` will be zero making the function to not revert as it should. 
Also, in the same block as the first deposit, `totalVotesAtAnchor` will still be zero and `anchorTime() - totalVotesAnchorTime` will also result in zero making the function to not revert as it should. 
