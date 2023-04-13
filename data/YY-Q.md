# [L-01] Missing input validation in `redeem` function (Equity.sol)

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L275
## Impact
The redeem function in Equity.sol does not check if the provided shares parameter is greater than zero. 

## Proof of Concept
There is no checking for `uint256 shares`, it accept zero value in this parameter.

## Tools Used
VS Code

## Recommended Mitigation Steps
Implement input validation such as `require(shares > 0, "Shares must be greater than zero");`

---

# [L-02] Missing input validation in `splitChallenge` function (MintingHub.sol)
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L156

## Impact
There is no validation on the splitOffAmount parameter. An attacker could potentially provide a malicious value, causing unexpected behavior in the contract. 

## Proof of Concept
There is no validation for splitOffAmount parameter

## Tools Used
VS Code

## Recommended Mitigation Steps
Implement input validation such as require()
e.g 
`require(splitOffAmount > 0 && splitOffAmount < challenge.size, "Invalid splitOffAmount value");`

--- 

# [L-03] Anyone can `end` function  (MintingHub.sol)
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L252

## Impact
Detailed description of the impact of this finding.

## Proof of Concept
Although some prevention have been set in this function, there is no checking for who call this function.
Based on the code of `end` function, it allowed everyone to call it.
```
function end(uint256 _challengeNumber, bool postponeCollateralReturn) public {
```

The attacker could prematurely end a challenge, resulting in the transfer of collateral to an unintended recipient, potentially the attacker themselves. 

## Tools Used
VS Code

## Recommended Mitigation Steps
Implement access control for this function.