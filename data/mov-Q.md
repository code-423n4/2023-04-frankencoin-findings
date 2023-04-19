** There's a bug in the function `restructureCapTable` in the Equity contract here: https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L309.
The function receives an array `addressesToWipe` and should iterate over it and burn the balances of the addresses in it. However there's a bug and the iteration variable `i` is not used in the loop, instead only the first element of the array is used as can be seen in this line:
`address current = addressesToWipe[0];`
I don't believe this is a critical bug because it can be mitigated in "production" by calling this function multiple times, each time with an array of size one with a different address. This however could be a costly and lengthy process.


** According to the documentation - https://docs.frankencoin.com/positions/open there should be a seven day cooldown period when creating a position. However there is a public `openPosition` function here: https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L88 which allows users to set an arbitrary amount in `_initPeriodSeconds` parameter. This parameter is later checked in the Position constructor, but the check is for three days which doesn't match the documentation. 


** It is possible to continuously bump the bid end time and postpone the challenge forever, in contrast with what the comments suggest here: https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L218. The bid doesn't necessarily have to "double" every two days.
If a challenge doesn't have any bids with a bid size over 0, then it is possible to keep bidding a `_bidAmountZCHF = 0` bid. In this line: https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L216 the bid amount is tested but if there are no previous bids or previous bids were 0, then this condition won't apply and the call will not be reverted. Instead, if we are close to the end of the challenge, this will bump the challenge end time. This will allow a malicious actor (the position owner for example) to continuously bump the challenge and not let it end. The challenger could mitigate this by generating a small bid but there is probably no nice GUI for this and only a sophisticated challenger will be able to understand why their challenge isn't ending, plus their bid to unlock the process will require a small fee. Effectively this means that most position owners are in danger of having their position frozen and challengers are in danger of having their challenge not end for a long time until a sophisticated white hat could figure out what's happening and unlock their position with a small bid.
To mitigate this I suggest not allowing bids with zero value. I don't believe there's any case for allowing those in the system anyways. 
