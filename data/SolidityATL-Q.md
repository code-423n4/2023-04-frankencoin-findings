## Title: [NC] Allowing infinite allowances can be harmful to users

## Links to affected code

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L125-L135

## Impact

This has been a long debated discussion in he Defi space. ultimately it is [harmful to allow infinite allowances in your protocol](https://kalis.me/unlimited-erc20-allowances/). In the event that an exploit occurs, all of users token funds are at risk as apposed to a limited approved amount. Additionally the [recent sushi swap hack](https://cryptoslate.com/sushiswap-token-allocation-exploit-drains-3-3m-as-users-urged-to-revoke-token-allowances-immediately/) would not have been possible if the users had not been able to allow for infinite approvals

## Proof of Concept

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L59-L64


1. Before opening a position a user must set allowances for both the ZCHF and collateral tokens


https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L125-L135


2. The ERC20 implementation that Frankencoin uses, allows users the ability to set Infinite allowances

## Tools Used

Manual review

## Recommended Mitigation Steps

Prevent unlimited approvals for Frankencoin ERC20 tokens. It's best to support the approval-spend flow for atomic transactions rather than allow infinite approvals.




## Title: [L] No minimum collateral enforced when opening a new position 

## Links to affected code

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L50-L71

## Impact

The Frankencoin docs state that when opening a position there should be a minimum initial collateral of ~5000 ZCHF. There is no minimum collateral enforced on the contract and it is possible to open a position with zero collateral
## Proof of Concept

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L50-L71

1. User creates a position with zero collateral 

## Tools Used

Manual Review

## Recommended Mitigation Steps

Enforce a minimum collateral by adding an addition `require` statement in the constructor of `Position.sol` or update the docs to mention this value can be zero




## Title: [L] Initialization Period in docs do not match the contract

## Links to affected code

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L50-L71
`require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values`

## Impact

When opening a position the docs mention a 7 day initialization period. In the code the minimum accepted initialization period is 3 days.
## Proof of Concept

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L50-L71 

## Tools Used

Manual Review

## Recommended Mitigation Steps

Update the docs to reflect the initial period of 3 days or update the code to match the doc