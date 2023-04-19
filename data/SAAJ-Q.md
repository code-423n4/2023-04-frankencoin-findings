
# Low Risk and Non-Critical Issues

# [L-01] Pragma Floating (10 Instances)
Locking pragma version ensures contracts are not being deployed on an outdated compiler version.

Link to the code:
1.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L2
2.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L2
3.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L4
4.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L2
5.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L12
6.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L5
7.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L2
8.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L2
9.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L3
10.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L9

# [L-02] Outdated Solidity Version (10 Instances)
Outdated version of solidity is used for all contracts in scope.
Recent version should be used to avoid known bugs that can impact the project negatively.


Link to the code:
1.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L2
2.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L2
3.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L4
4.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L2
5.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L12
6.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L5
7.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L2
8.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L2
9.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L3
10.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L9


# [L-03] Missing checks for address(0x0) when assigning values to address state variables (03 Instances)

Zero-address check should be used in the constructors, to avoid the risk of setting a storage variable as address(0) at deploying time.

Link to the code:

1.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L50
2.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L54
3.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L26


# [L-04] Missing event for important parameter change (10 Instances)

Important parameter or configuration changes should trigger an event to allow being tracked off-chain.

Link to the code:
1.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L97
2.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L132
3.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L177
4.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L177
5.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L202
6.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L263
7.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L329
8.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L124
9.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L281
10.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L83

# [N 1] NatSpec is incomplete (11 Instances)

Link to the code:
1.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L169
2.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L181
3.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L193
4.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L202
5.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L286
6.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L292
7.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L281
8.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L287
9.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L190
10.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L225
11.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L266


# [N-02] Function writing that does not comply with the Solidity Style Guide

All Contracts
Order of functions should  help readers identify which functions they can call and to find the constructor and fallback definitions easier.

Functions should be grouped according to their visibility and ordered as mentioned in the article i.e.;
constructor
external
public
internal
private
within a grouping, place the view and pure functions last
