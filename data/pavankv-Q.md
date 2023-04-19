## 1 . check of  minBid(challenge) must be first position in function:-

**Summary**
In [bid()](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L199) first transfer the challengebid to challenger and then in line [216](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L216 ) checks the after all computation so it's good practice to check before transfer the challengebid and it's save gas also by avoid computation even the _bidAmountZCHF is lesser than minimumbid.

code snippet:-
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L216 

## 2. Instead of minterOnly() modifier use directly (if else) or (require) to check minter  .

**Summary :-**
In minterOnly() modifier calls isMinter() and isMinter(positions[msg.sender]) in if else statement to check whether the msg.sender is minter or not.

Modifiers can be used to:

Restrict access
Validate inputs
Guard against reentrancy hack

But [isMinter()](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L293) function only checks whether the msg.sender is minter or not return true or false . minterOnly() modifier checks true or false return by isMinter() function ,so instead of minterOnly() modifier we can use directly isMinter() in require or if else statement even it saves gas .

 code snippet:-
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L266
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L293

Recommendation :-
require(isMinter(),"not minter");
 if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();

Add above any one code in restricted function instead of minterOnly() modifier to save gas and reduce the code .



## 3 . Add alive() modifier in notifyChallengeSucceeded() function 

code snippet:-
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L329
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L366
