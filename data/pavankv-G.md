## 1.Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2).

Saves 21 gas per call

code snippet :-
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L132
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L159
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L227
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L249
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L292

## 2 . use Ternary operate to save gas :-

code snippet:-
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L250
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L121
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L160
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L184

## 3 .Multiple address mappings can be combined into a single mapping of an address to a struct, where appropriate
Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations.

saves 42 per access

code snippet:-
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L37
