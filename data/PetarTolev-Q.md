# Low Risk Findings Overview
| ID     | Finding                                                                                  | Instances  |
| ------ | ---------------------------------------------------------------------------------------- | ---------- |
| [L-01] | Missing zero-address check in the setter constructors, functions or initialize functions | 3          |
| [L-02] | Ownership May Be Burned                                                                  | 1          |
| [L-03] | Use Ownable2Step instead of Ownable contract                                             | 1          |

*Total: 5 instances over 3 issues*

# Non-critical Findings Overview
| ID     | Finding                                                                | Instances  |
| ------ | ---------------------------------------------------------------------- | ---------- |
| [N-01] | Contract files should define a locked compiler version                 | 10         |
| [N-02] | Instead of using require statements, only custom errors should be used | _          |
| [N-03] | Invalid Order of layout                                                | 50         |
| [N-04] | Missing NatSpec                                                        | 10         |

*Total: 70 instances over 4 issues*

---

# Low Risk Findings

## [L-01] Missing zero-address check in the setter constructors, functions or initialize functions

*3 Instances:*

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L54 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L50-L70 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L26-L31

## [L-02] Ownership May Be Burned
The ownership can be transferred to address(0), effectively burning the ownership.

*1 Instances:*
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L39-L43

```solidity
function setOwner(address newOwner) internal {
    address oldOwner = owner;
    owner = newOwner;
    emit OwnershipTransferred(oldOwner, newOwner);
}
```

### Recommendation

Implement a validation check to ensure that ownership is not transferred to address(0).

```diff
function setOwner(address newOwner) internal {
+   require(newOwner != address(0), "Invalid new owner: address(0)");
    address oldOwner = owner;
    owner = newOwner;
    emit OwnershipTransferred(oldOwner, newOwner);
}
```




## [L-03] Use Ownable2Step instead of Ownable contract

There is another OpenZeppelin Ownable contract [(Ownable2Step.sol)](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) that has a `transferOwnership` function, which is more secure due to the 2-stage ownership transfer.


# Non-critical Findings

## [N-01] Contract files should define a locked compiler version

Solidity version varies, is not the most recent. The version of Solidity used throughout the codebase varies, but it is not pinned, nor is it the latest stable version. Use latest solidity version 0.8.19

*10 Instances:*

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L2 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L2 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L4 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L2 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L12 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L5 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L2 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L2 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L3 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L9

## [N-02] Instead of using require statements, only custom errors should be used

## [N-03] Invalid Order of layout

To improve the readability and maintainability of Solidity code, it is recommended to follow a consistent layout and order in the contracts. The Solidity documentation provides ([Order of Layout](https://docs.soliditylang.org/en/v0.8.19/style-guide.html#order-of-layout), [Order of functions](https://docs.soliditylang.org/en/v0.8.19/style-guide.html#order-of-functions)) a suggested convention, which includes the order of pragma and import statements, interfaces, libraries, contracts, and the order of type declarations, state variables, events, functions, modifiers, and errors inside a contract. This can help avoid errors and inconsistencies and make the code more understandable for other developers.

- Pure/view functions should be in the end of the contract

*18 Instances:*

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L97-L110 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L127-L137 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L172-L212 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L225-L233 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L262-L270 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L290-L297 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L64-L70 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L102-L119 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L138-L146 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L204-L213 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L235-L240 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L181-L190 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L239-L241 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L120-L126 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L169-L171 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L181-L189 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L282-L284 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L360-L362

- Error declarations should be at the beginning of the contract

*14 Instances:*

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L214 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L92-L94 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L130 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L159 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L231-L233 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L45 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L103-L104 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L191 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L238 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L290 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L364 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L371 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L378 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L385

- Modifiers should be declared at the beginning of the contract, before functions

*7 Instances:*

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L266-L269 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L115-L118 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L49-L52 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L366-L369 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L373-L376 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L380-L383 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L387-L390

- Interfaces should be declared at the beginning of the file, after imports

*1 Instances:*

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L299-L315

- Struct declarations should be at the beginning of the contract

*1 Instances:*

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L39-L46

- The order of functions should be: constructor, receive function (if it exists), fallback function (if it exists), external, public, internal, private, view/pure

*9 Instances:*

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L112-L122 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L144-L148 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L157-L167 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L152-L157 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L165-L197 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L140-L179 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L199-L229 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L44-L53 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L67-L70

## [N-04] Missing NatSpec

Throughout the codebase there are several parts that missing NatSpec

*10 Instances:*

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L181 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L292 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L59 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L181 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L235 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L239 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L190 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L172 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L162 \
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L55
