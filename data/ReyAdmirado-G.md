| | issue |
| ----------- | ----------- |
| 1 | [state variables should be cached in stack variables rather than re-reading them from storage](#) |
| 2 | [can make the variable outside the loop to save gas](#) |
| 3 | [Ternary over if ... else](#) |
| 4 | [Use assembly to check for address(0)](#) |
| 5 | [before some functions we should check some variables for possible gas save](#) |
| 6 | [Non-strict inequalities are cheaper than strict ones](#) |

## 1. state variables should be cached in stack variables rather than re-reading them from storage

Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 


`minters[_minter]` should be cached before #L294 because the is a high possibilty of it being read twice from storage
- [Frankencoin.sol#L294](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L294)

possible 97 gas save with risking loss of only 3 gas. if the check passes we save 97 for the second `minted` read and if it fails we just lose 3 gas. cache it before this line
- [Position.sol#L349](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L349)

`minted` should be cached before #L141 because its being read at least twice. first read is in the condition in #L141 in case the condition succeeds its gonna be read in the #L142 which could be prevented by using the cached version. in case the condition in #L141 fails its gonna be read in #L149. 
- [Position.sol#L141-L150](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L141-L150)


## 2. can make the variable outside the loop to save gas

make the variable outside the loop and only give the value to variable inside

`current` 
- [Equity.sol#L193](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L193)
- [Equity.sol#L313](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L313)

`j`
- [Equity.sol#L196](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L196)


## 3. Ternary over if ... else

Using ternary operator instead of the if else statement saves gas. 
the placed that it make sense are listed here

- [Position.sol#L121-L125](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L121-L125)
- [Position.sol#L160-L164](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L160-L164)
- [Position.sol#L184-L188](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L184-L188)
- [Position.sol#L250-L254](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L250-L254)

- [Frankencoin.sol#L141-L144](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L141-L144)
- [Frankencoin.sol#L207-L211](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L207-L211)
- [Frankencoin.sol#L282-L287](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L282-L287)


## 4. Use assembly to check for address(0)

saves 6 gas per instance

- [ERC20.sol#L152](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L152)
- [ERC20.sol#L180](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L180)

- [ERC20PermitLight.sol#L56](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L56)


## 5. before some functions we should check some variables for possible gas save

before transfer we should check for amount being 0 so the function doesnt run when its not gonna do anything

check `amount`
- [Position.sol#L228](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L228)
- [Position.sol#L253](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L253)
- [Position.sol#L269](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L269)

- [MintingHub.sol#L110](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L110)
- [MintingHub.sol#L129](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L129)
- [MintingHub.sol#L142](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L142)
- [MintingHub.sol#L284](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L284)
- [Frankencoin.sol#L283](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L283)


`freedAmount - _amountExcludingReserve`
- [Frankencoin.sol#L254](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L254)

`reserveLeft`
- [Frankencoin.sol#L285](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L285)


## 6. Non-strict inequalities are cheaper than strict ones

In the EVM, there is no opcode for non-strict inequalities (>=, <=) and two operations are performed (> + = or < + =).
consider replacing >= with the strict counterpart > and add `- 1` to the second side

- [Position.sol#L53](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L53)
- [Position.sol#L110](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L110)
- [Position.sol#L184](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L184)
- [Position.sol#L305](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L305)
- [Position.sol#L307](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L307)
- [Position.sol#L374](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L374)

- [MintingHub.sol#L109](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L109)
- [MintingHub.sol#L171](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L171)
- [MintingHub.sol#L172](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L172)
- [MintingHub.sol#L201](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L201)
- [MintingHub.sol#L218](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L218)
- [MintingHub.sol#L255](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L255)

- [Equity.sol#L136](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L136)
- [Equity.sol#L244](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L244)
- [Equity.sol#L247](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L247)

- [Frankencoin.sol#L141](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L141)
- [Frankencoin.sol#L282](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L282)
- [Frankencoin.sol#L294](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L294)

- [ERC20PermitLight.sol#L30](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L30)

- [StablecoinBridge.sol#L50](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L50)
- [StablecoinBridge.sol#L51](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L51)