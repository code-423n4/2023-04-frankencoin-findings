
# Gas Optimization Report

This report focuses on Frankencoin contest, in context of various improvements that can be made in terms of gas cost.

Some of the opportunities identified for improving gas efficiency for the code in scope of Frankencoin are categorised into 04 main areas; with further multiple instances in each of the category.


# Summary

 [G-01] Multiple mappings can be combine into a single one (06 Instances)
 [G 02] Declare Public library as internal library (02 Instances)
[G-03] Immutable has more gas efficiency than constant (06 Instances)
[G-04] Use != 0 instead of > 0 for unsigned integer comparison (02 Instances)

 
# [G-01] Multiple mappings can be combine into a single one (06 Instances)

When multiple mappings are used in same function, it’s better to combined them into a single mapping using a struct.

Combined mapping reduces storage slot per mapping and also are cheaper in terms of associated stack operations calculation carried out.

Link to the Code:
1.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L83
2.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L88
3.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L45
4.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L50
5.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L43
6.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L45


# [G 02] Declare Public library as internal library (02 Instances)

External call to a public library function is very expensive for reference checks this article. When library has only internal functions it can be used as internal library.

Link to the code:

1.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L9
2.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L9


# [G-03] Immutable has more gas efficiency than constant (06 Instances)

Using immutable instead of constant, save more gas due to avoiding storage access for state variables.

Variable values are set through constructor when using immutable, which also eliminates the need of assigning initial values to state variable making them more efficient in terms of gas cost.

Link to the Code:

1.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L20
2.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L26
3.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L39
4.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L46
5.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L51
6.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L25


# [G-04] Use != 0 instead of > 0 for unsigned integer comparison (02 Instances)

Link to the Code:
1.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L381
2.	https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L114

