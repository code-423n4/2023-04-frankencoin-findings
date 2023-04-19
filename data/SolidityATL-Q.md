title: [NC] Allowing infinite allowances can be harmful to users

## Links to affected code

https://github.com/code-423n4/2023-04-frankencoin/blob/b94acfc6c8dd9b9c42db212d9455e3493b9bff2f/contracts/ERC20.sol#L125-L135

## Impact

This has been a long debated discussion in he Defi space. ultimately it is [harmful to allow infinite allowances in your protocol](https://kalis.me/unlimited-erc20-allowances/). In the event that an exploit occurs, all of users token funds are at risk as apposed to a limited approved amount. Additionally the [recent sushi swap hack](https://cryptoslate.com/sushiswap-token-allocation-exploit-drains-3-3m-as-users-urged-to-revoke-token-allowances-immediately/) would not have been possible if the users had not been able to allow for infinite approvals

## Proof of Concept

https://github.com/code-423n4/2023-04-frankencoin/blob/b94acfc6c8dd9b9c42db212d9455e3493b9bff2f/contracts/MintingHub.sol#L59-L64

1. Before opening a position a user must set allowances for both the ZCHF and collateral tokens

https://github.com/code-423n4/2023-04-frankencoin/blob/b94acfc6c8dd9b9c42db212d9455e3493b9bff2f/contracts/ERC20.sol#L125-L135

2. The ERC20 implementation that Frankencoin uses, allows users the ability to set Infinite allowances

## Tools Used

Manual review

## Recommended Mitigation Steps

Prevent unlimited approvals for Frankencoin ERC20 tokens. It's best to support the approval-spend flow for atomic transactions rather than allow infinite approvals.