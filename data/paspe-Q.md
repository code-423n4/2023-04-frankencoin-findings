# [QA-1] Use better order of variables to increase readability
 https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L366  https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L238 
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L214 
 Errors should be on top of the contract but not between methods. Modifiers are also hard to follow since they are all over the contract. They should be below events declaration. Here is a reference from official Solidity doc for the ordering the layout of the contract and best practices 
https://docs.soliditylang.org/en/v0.8.12/style-guide.html?highlight=order#order-of-layout 

# [QA-2] In several places the number `1000_000` is not written as 1 million. You can use the underscore to look like 1 million `1_000_000` 
  https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L122 
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L166 

# [QA-3] Values helping to prevent division rounding can be extracted to constant variables and to have proper name. And when they are repeated it exists some danger for copy/pasting errors
  https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L124     
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L239 
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L166 
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L239 
 It can be named like: `uint valueToPreventDivisionRounding = 1_000_000;`
# [QA-4] removing from array with delete can product used slot
`delete challenges[_challengeNumber];` can be replaced with `.pop()` like that `challenges[_challengeNumber].pop();`

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L213 
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L275 
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L283 
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L155 

# [QA-5] Use stable pragma statement 
Using a floating pragma statement ^0.8.0 is discouraged as code can compile to different bytecodes with different compiler versions. Use a stable pragma statement to get a deterministic bytecode. It is important to lock the pragma (for example, not using ^ in pragma solidity 0.8.10) to prevent contracts from being accidentally deployed using an older compiler with unfixed bugs.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L2 

# [QA-6]  Different pragma directives are used
Having `pragma solidity >=0.8.0 <0.9.0;` is not good practice also.

It is mistake also to use different pragma versions across different files, which can cause inconsistency and introduce security issues. It is better to use one Solidity compiler version across all contracts instead of different versions with different bugs and security checks.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L4  

# [QA-7] Don’t use magic numbers `10000`, `1000`, etc.

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L211
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L268 
Declare a constant with meaningful name