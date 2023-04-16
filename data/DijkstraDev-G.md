#### [G-01] Remove the balance check required for the transfer.
In version ^0.8.0, it reverts in case of under/overflow. In this case, if the user does not have enough balance (balances[sender] < amount), the operation balances[sender] -= amount will fail. Additionally, this allows correct transfers to use less gas.
[ERC20.sol#L155](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/ERC20.sol#L155)
[ERC20.sol#L156](https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/ERC20.sol#L156)