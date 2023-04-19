# 1: GENERATE PERFECT CODE HEADERS EVERY TIME					

Vulnerability details

## Context:

We recommend using a header for Solidity code layout and readability

> ***Example***

 /*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/


For reference, see https://github.com/transmissions11/headers

NOTE: ERC20PermitLight.sol already applies this recommendation.

## Proof of Concept

> ***Contracts***

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol

## Tools Used

Manual Analysis


# 2: WORD & TYPING TYPOS

Vulnerability details

## Context:

Word & typing typos

## Proof of Concept

> ***File: Frankencoin.sol*** 

arbitraty can be changed to arbitrary in the following comment.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L100


elses can be changed to else in the following comment.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L260


> ***File: Frankencoin.sol*** 

creterion can be changed to criterion in the following comment.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L358


> ***File: Equity.sol*** 

proporational can be changed to proportional in the following comment.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L34


## Tools Used

Manual Analysis


