## 1 . Directly call challenge.bid in minBid() to saves:-
In [bid](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L199)() function [calls](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L216) the minBid() public function with [challenge](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L200) variable which is initialise by storage only , in  minBid() public function uses challenges storage variable array  to search given challenge variable which is already found in bid() only then it calls [minBid()](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L188) internal function , in this it will again initialise from storage only, to get challenge.bid .But we can absorb here challenge variable  which is already initialise from storage in bid() and try search again and initialise in minBid() functions from storage again  .

 So Sload consumes 200-800 gas  each time , if challenges[] arrays grows large then it consumes 800 gas . 

**Recommedation :-**
try to directly call minBid(challenge.bid) in bid() and in internal minBid(challenge.bid) also which saves the gas cost .

**change to :-**
in bid() 
minBid(challenge.bid)

in minBid( uint challengeBid) public  function ///
minBid(challenge.bid)

in minBid(uint challengeBid) internal function ///
minBid(challenge.bid) // to calculate the minimum bid .


code snippet:-
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L216




## 2.Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2).

Saves 21 gas per call

code snippet :-
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L132
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L159
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L227
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L249
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L292

## 3 . use Ternary operate to save gas :-

code snippet:-
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L250
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L121
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L160
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L184

## 4 .Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

saves 42 per access

code snippet:-
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L37
