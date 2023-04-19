## 0. `Equity.restructureCapTable` does not iterate the array `addressToWipe`
https://github.com/code-423n4/2023-04-frankencoin/blob/e13bce4f0f9756d0e6145ad2d1992494f983dce5/contracts/Equity.sol
Change the line to `address current = addressesToWipe[i];`.

## 1. Check arguments before interacting with external contracts
`require` should be checked at the beginning of the function `MintingHub.openPosition`.
https://github.com/code-423n4/2023-04-frankencoin/blob/e13bce4f0f9756d0e6145ad2d1992494f983dce5/contracts/MintingHub.sol#L109

## 2. Delete `override` in state variables
https://github.com/code-423n4/2023-04-frankencoin/blob/e13bce4f0f9756d0e6145ad2d1992494f983dce5/contracts/Frankencoin.sol#L31
https://github.com/code-423n4/2023-04-frankencoin/blob/e13bce4f0f9756d0e6145ad2d1992494f983dce5/contracts/Position.sol#L35-L36