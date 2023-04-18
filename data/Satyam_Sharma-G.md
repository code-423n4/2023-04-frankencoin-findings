# 1. gas optimization
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

# 2.gas optimization

Under frankencoin.suggestMinter totalsupply() is checked two times, which can be expensive as Every time totalSupply() is called, it requires a read operation from the blockchain state which consumes gas.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84-L85
```
 if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
```

Check under one condition is sufficient since is uses && operator which only proceeds the code further if both the conditions are right.
