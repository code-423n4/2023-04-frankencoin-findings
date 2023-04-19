# Audit Report

## Summary

### Low Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | `registerPosition()` should be done at the end of the function | 1 | 
| [L&#x2011;02] | Use `safeERC20.safeApprove()` instead of `approve()`| 2 | 


### Non-critical Issues
| |Issue|Instances|
|-|:-|:-:|
| [N&#x2011;01] | `initPeriod` should be 7 days | 1 | 
| [N&#x2011;02] | Interfaces should be in seperate files | 2 | 
| [N&#x2011;03] | Constants should be used rather than magic numbers | 3 | 


### [L&#x2011;01] `registerPosition()` should be done at the end of the function

### Impact
This does not pose any immediate risks, but it is better to register the position after all the other effects are done.

#### Findings
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L107

#### Recommended Mitigation Steps
`registerPosition()` should be called at the end of `openPosition()` and before `return`.


### [L&#x2011;02] Use `safeERC20.safeApprove()` instead of `approve()`

### Impact
Note that `approve()` will fail for certain token implementations that do not return a boolean value . Hence it is recommend to use `safeApprove()`.

#### Findings
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L108
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L221

#### Recommended Mitigation Steps
Update the code with `safeApprove()`.


### [N&#x2011;01]  `initPeriod` should be 7 days

#### Impact
It's recommended to set `initPeriod` to a constant value and specially because elsewhere the period is assumed to be a week. According to the docs it should be `7 days`. 

#### Findings
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L53
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L88


#### Recommended Mitigation Steps
Hardcode the passed parameter with `7 days` in `openPosition()`.

### [N&#x2011;02]  Interfaces should be in seperate files

#### Impact
Move the interfaces to a seperate file for better readability.

#### Findings
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L299

### [N&#x2011;03]  Constants should be used rather than magic numbers

#### Findings
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L118
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L205
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L166
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L239
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L211
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L247
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L268
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L265
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L122
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L124

#### Recommended Mitigation Steps
Define constant variables for the findings mentioned above.