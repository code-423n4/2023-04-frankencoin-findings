## 1.Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2).

Saves 21 gas per call

code snippet :-
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L132
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L159
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L227
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L249

## 2 . use Ternary operate to save gas :-

code snippet:-
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L250
