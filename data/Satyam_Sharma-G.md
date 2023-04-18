under bid function expectedSize should be used instead using challenge.size which saves gas , since using challenge.size directly from storage is expensive and since it checks whether expectedSize equals to challenge.size then only it proceeds to the below code therefore we can use expectedSize instead challenge.size.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L208-L212

```
 if (challenge.position.tryAvertChallenge(challenge.size, _bidAmountZCHF)) {//@audit using expectedSize instead challenge.size save us gas 
            // bid was high enough, let bidder buy collateral from challenger
            zchf.transferFrom(msg.sender, challenge.challenger, _bidAmountZCHF);
            challenge.position.collateral().transfer(msg.sender, challenge.size);//@audit using expectedSize instead challenge.size save us gas 
            emit ChallengeAverted(address(challenge.position), _challengeNumber);
            delete challenges[_challengeNumber];
```
