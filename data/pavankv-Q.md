## 1 . check of  minBid(challenge) must be first position in function:-

**Summary**
In [bid()](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L199) first transfer the challengebid to challenger and then in line [216](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L216 ) checks the after all computation so it's good practice to check before transfer the challengebid and it's save gas also by avoid computation even the _bidAmountZCHF is lesser than minimumbid.

code snippet:-
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L216 

## 2 . Add alive() modifier in notifyChallengeSucceeded() function 

code snippet:-
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L329
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L366
