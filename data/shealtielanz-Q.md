`Low Risk & Non Critical Findings`
# Report
## Low Risk Findings 
**Low Risk Findings List**
| Number | Issue Details | Instances |
|-----:|----|-----|
| L-01 | Missing `Reentrancy_Guard` on Critical functions | 1 |
| L-02 | Use of  Unlocked `Pragmas` to specify complier versions | 9 |
| L-03 | Ownership can be lost | 1 |
| L-04 | The `Allowance` Function can be sandwiched  via racing conditions | 1 |

> Total ~ 4 Issues


# L-01. Missing `Reentrancy_Guard` Modifiers on Critical functions
## Summary 
> Likelihood ~ High, as reentrancy is a very common attack vector and easily exploitable
There is no Reentrancy Guard in certain functions in the protocol which can be reentered on certain conditions, and can drain funds and collateral by the owner.
## Vulnerability Details 
Some functions within the protocol can be reentered, thereby allowing for drainage of funds, below is an example 

``` solidity 
268. function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {
269.          IERC20(collateral).transfer(target, amount);
270.         uint256 balance = collateralBalance();
271.     if (balance < minimumCollateral){
272.            cooldown = expiration;
273.       }
274.        emitUpdate();
275.      return balance;
276.    }
  ```
      
## Impact
The Reentrancy Bug is the most common security risk in the web3 space but the risks are great, where all the funds in a contract can be drained via a fallback function.
## Recommended Mitigation Steps
Make sure the above functions is following the Checks-Effects-Interactions pattern or add a nonReentrant modifier to it.


# L-02 ~ Use of `Unlocked/Floating` Pragma while specifying the complier version

## Summary 
Use of `Floating` or `Unlocked` Pragma eg, Pragma directives with the carrot sign `^`in a codebase allows for testing and deploying with a different complier versions, which can cause undefined behaviour when deployed on the mainet. 

## Vulnerability Details
> Testing & Deploying of Smart Contracts with different complier versions.

The bug arise due to the Carrot sign `^` used in the pragma directive, 
``` solidity 
pragma solidity ^0.8.0;
```
which allows the contract to be tested with a different complier version from the one that it would be deployed with on the mainet, as one can run tests with a different complier version that has different features and security checks but the newer one that is used for deployment may support a different set of features and security checks, thus this mismatch between the testing and deployment is allowed by the use of this pragma (which can cause the smart contracts to behave in a way unintended by the developers), 
hence is not recommended to be used

## Code Snippet

### Instances - All Contracts in the scope

- [Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol)
- [MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol)
- [Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol)
- [FrankenCoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol)
- [ERC20](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol)
- [ERC20PermitLight.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol)
- [StableCoinBrigde.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol)
- [PositionFactory.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol)
- [Ownable.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol)
- [MathUtil.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol)
## Impact
Use of different complier versions for testing and deployment can enable certain bugs to slip during testing due to the different bugs and bug fixes of different complier versions, which can cause *Undefined Behaviour in the Applications of the Protocol* when deployed on the mainet and also missing checks that could bring about *Over/Under Flow Issues* Which arise due to missing SMT(SafeMaths) checks.

## Tools Used
```
Manual Review & Security Pitfalls & Best Practices (https://youtu.be/IXm6JAprhuw)
```

# Recommended Mitigation Steps
**Insure the use of a specific Complier Version**
> Use of Locked Pragma for specifying the complier version to be used for testing and deployment.

``` solidity 
1. // SPDX-License-Identifier: MIT
2. pragma solidity 0.8.0;
```
That is by removing the carrot sign `^`

# L-03 Ownership can be lost
## Summary 
Changing critical addresses in contracts should be a two-step process where the first transaction (from the old/current address) registers the new address (i.e. grants ownership) and the second transaction (from the new address) replaces the old address with the new one (i.e. claims ownership). This gives an opportunity to recover from incorrect addresses mistakenly used in the first step. If not, contract functionality might become inaccessible. (see here and here)
## Vulnerability Details 
This arises in the  [Ownable.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol)
The `setOwner` function which does swift transfer of ownership without making sure that someone owns the address.
``` solidity 
39  function setOwner(address newOwner) internal {
40       address oldOwner = owner;
41        owner = newOwner;
42        emit OwnershipTransferred(oldOwner, newOwner);
43        }
```
> `Ownable.sol` is inherited in `Position.sol`

## Impact
Functionalities inside the contract will be inaccessible if there is a mistake in Changing the ownership of a contract.
> Likelihood ~ Low

## Recommended Mitigation Steps
> Use a 2 structure transferOwnership which is safer.
safeTransferOwnership, use it is more secure due to 2-stage ownership transfer.
Use [Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol)

# L-04 The `Allowance` Function can be sandwiched  via racing conditions
## Summary 

## Vulnerability Details 
Example in [ERC20](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol) the `allowanceInternal(address`
``` solidity
function allowanceInternal(address owner, address spender) internal view virtual returns (uint256) {
        return _allowances[owner][spender];
    }
```
> Exploit Scenerio 

The race condition that happens the most on the network today is the race condition in the ERC20 token standard. The ERC20 token standard includes a function called 'approve' which allows an address to approve another address to spend tokens on their behalf. Assume that Alice has approved Eve to spend n of her tokens, then Alice decides to change Eve's approval to m tokens. Alice submits a function call to approve with the value n for Eve. Eve runs a Ethereum node so knows that Alice is going to change her approval to m. Eve then submits a tranferFrom request sending n of Alice's tokens to herself, but gives it a much higher gas price than Alice's transaction. The transferFrom executes first so gives Eve n tokens and sets Eve's approval to zero. Then Alice's transaction executes and sets Eve's approval to m. Eve then sends those m tokens to herself as well. Thus Eve gets n + m tokens even thought she should have gotten at most max(n,m).
## Impact
The `Allowance` Function allows a maliciously person to do a double spend, if the owner of the tokens wants to reduce he/her allowance. 
> Likelihood ~ Medium
## Recommended Mitigation Steps
**Use safeIncreaseAllowance() and safeDecreaseAllowance() from OpenZeppelin’s SafeERC20 implementation to prevent race conditions from manipulating the allowance amounts.**


`Non-Critical Findings`

## Non Critical Issues

**Non Critical Issues List**

| Number | Issue Details | Instances |
|-----:|----|-----|
| NC-01 | Incomplete `Natspecs` to aid in code readability | 9 |
| NC-02 | Arrangement of `Modifiers`, `Error statements`, `Events` and `Functions` Do not comply with Solidity Security Style | 5 |
| NC-03 | Missing `Events` for State Changing Methods | 1 |
| NC-04 | Update `Import` Usage for a more modern codebase style | 4|
| NC-05 | Use underscores for number literals | 1 |

> Total ~ 5 Issues



# NC-01 Incomplete `Natspecs` to aid in code readability and Audits

## Summary 
**Incomplete Natspec and documentation across all the codebase in the protocol which causes confusion and lack of understanding when understanding most of the logic of functions in the protocol.**
## Vulnerability Details
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
Most external methods in the codebase lack complete NatSpec documentation some functionn almost always lack @param and @return explanation. Short example is 👇🏽

``` solidity 
Position.sol 
constructor(address _owner, address _hub, address _zchf, address _collateral, 

        uint256 _minCollateral, uint256 _initialLimit, uint256 initPeriod, uint256 _duration,

        uint256 _challengePeriod, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) {

        require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values

```
        
        
most of the parameters or the return variable are not listed in the NatSpec and so many other functions in the protocol.

## Impact
This makes it harder to read and understand both the logic and the assumptions of the code in the protocol causing slippage of bugs Unknown to the developers and auditors.
## Recommended Mitigation Steps
Go through all methods, codebase and logic of the Protocol  and make sure they have a complete and proper documentation.



# NC-02  Arrangement of `Modifiers`, `Error statements`, `Events` and `Functions` Do not comply with the Solidity Security Style
## Summary 
**In Solidity there is a recommended Writing style to aid readability and Auditing of Smart Contracts, which should be followed although the Impact may not be high but it affect how the contracts are being audited and if a code is hard to audit it causes the auditors to miss certain bugs which might and will affect the protocol in future.**
## Vulnerability Details 
> Instances ~ 4
The following contracts do not adhere to both the Recommended code layout and writing style of Solidity 
- [Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol)
- [MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol)
- [FrankenCoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol)
- [Ownable.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol)
## Impact
The recommended Writing style and code layout aid auditors to find bugs easily and with more precision, not following it causes Inconsistent audits and makes a codebase hard to read.
## Recommended Mitigation Steps

> Code Layout 👇🏽

Layout contract elements in the following order: Pragma statements, Import statements, Interfaces, Libraries, Contracts. Inside each contract, library or interface, use the following order: 
- Type declarations, 
- State variables, 
- Events, 
- Functions
> Ordering Style 👇🏽

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: 
- constructor, 
- receive function (if exists), 
- fallback function (if exists), 
- external, 
- public, 
- internal, 
- private. 
- Within a grouping, place the view and pure functions last.

# NC-03 Missing event emissions in state changing methods
## Summary 
**Events help non-contract tools to track changes, and events prevent users from being surprised by changes also helps the developers get notified easily when there is any undefined behaviour or attack in the the protocol,**
## Vulnerability Details 
Some state changing functions and parameters do not emit events 
example 👇🏽
-  [PositionFactory.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol)

## Impact
lack of events and emissions of this events can cause alot of unexpected changes and activities that might take both the developers and users unawares

## Recommended Mitigation Steps
It's a best practice to emit events on every state changing method for off-chain monitoring. The setBaseURI and setGenerated methods in HoneyJar are missing event emissions, which should be added.

# NC-04 Update `Import` Usage for a more modern codebase style
## Summary 
There is no need for unused Import statements in certain contracts and should be removed.
## Vulnerability Details 
The following contracts make use of imports that are not used at all.
- [MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol)
- [FrankenCoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol)
- [Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol)
- [Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol)
- [StableCoinBrigde.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol)
## Impact
There is no use of Import statements that don't matter in a codebase, it is rather removed as to avoid confusion and time wasting reading such imports.
## Recommended Mitigation Steps
Clear `Import` Statements in the above contracts that are not used.

# N-05 Use underscores for number literals
## Summary 
There are occasions where certain numbers have been hardcoded, either in variable or in the code itself. Large numbers can become hard to read.

## Vulnerability Details 
In the [MathUtil.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol)
``` solidity

11.   uint256 internal constant THRESH_DEC18 =  10000000000000000;

```

## Impact
Lack of Use of underscores for number literals makes such numbers hard to work with and prompts for errors when handling  such hard coded literals.

## Recommendation:

Consider using underscores for number literals to improve its readability.
Example 👇🏽
``` solidity

11.   uint256 internal constant THRESH_DEC18 =  1_000_000_000_000_000_0;

```







