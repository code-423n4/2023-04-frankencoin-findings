# 1. Adding a malicious message when suggesting a minter can target the reviewers with Web2 attacks

## Impact

LOW

- It can be assumed that the reviewers of suggested minters are invested in cryptocurrency and rather financially well off. Therefore it can be concluded that they are an audience worth attacking for a malicious actor. In context of Web3 attack vectors one should not ignore that Web2 attack vectors are still relevant.

- The \_message parameter of the suggestMinter() function in Frankencoin.sol (https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L83) allows the submission of additional information regarding the suggested minter. A malicious actor may for example include links that lead to phishing attacks, malware infection, ransomware attacks, scams and frauds.

- Although suggesting a minter costs an attacker the minimum fee to be reviewed (if totalSupply > 0), the highly targeted audience of minter reviewers may still be worth the investment for the attacker. In an extreme case a reviewer could be pressed to act against the interests of the Frankencoin project and its Frankencoin Pool Shares holders e.g. to not veto a suggested minter which could introduce a malicious minting strategy and drain the project.

## Tools Used

Manual review

## Recommended Mitigation Steps

Either don't allow links to be passed in the \_message parameter of the suggestMinter() function or sanitize submitted links and inform the reviewers about the potential risks of included links BEFORE the review. It is assumed that the review of suggested minters will be enabled by the Frankencoin project through some web frontend. This would available to eligible reviewers to make the review process convenient (not requiring to interact with on-chain information and contracts directly). Here a warning could be displayed that needs to be acknowledged before the review can begin.

---

# 2. Setting \_minApplicationPeriod in Frankencoin.sol constructor needs sanity check

## Impact

LOW

- The \_minApplicationPeriod parameter in Frankencoin.sol constructor is not sanity checked: https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L59

- The missing check could lead to it being set to low or too high. It needs to be set in seconds but could accidentally be set in higher or lower units (days/milliseconds) or correctly in seconds but too low or too high. There is no proper NatSpec in place for the constructor which increases the risk that the comments on the constructor are overlooked.

- Worst case of setting it too low or even to 0: This would lead to minters being active instantly (if \_minApplicationPeriod is 0) or to a review period quickly passing and automatically activating the minter before any review can be done and a potential negative vote can be cast. This puts the project at risk to e.g. minting an infinite number of Frankencoins via a malicious minter.

- Worst case of setting a too high value: With a very high value the time could be so long that minters would never pass the \_minApplicationPeriod. This fails the idea of the project by allowing others to introduce new minters and it would be stuck with the initially introduced 2 trusted minters by the Frankencoin team. This can be considered as DoS for minter suggestion.

## Tools Used

Manual review

## Recommended Mitigation Steps

Introduce a sanity check in the Frankencoin.sol constructor for \_minApplicationPeriod e.g. the \_minApplicationPeriod > 3 days or > 7 days. I have seen both numbers in the code so not sure which one would be the correct sanity check.

---

# 3. Missing sanity checks for constructor arguments of StablecoinBridge.sol

## Impact

LOW

The constructor in https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L26 does not check for the passed arguments. This could in the worst case open the door for exploits or in the most positive case require a redeploy of the StablecoinBridge contract.

## Tools Used

Manual review

## Recommended Mitigation Steps

- Add validation / sanity checks for all constructor arguments

- Passed addresses should at least be checked or not being the zero address

- The passed limit\_ parameter should be checked for being in a valid range (not 0, not too low, not too high).

---

# 4. restructureCapTable function in Equity.sol is only wiping for first entry in addressesToWipe

## Impact

LOW

- The index for retrieving the address to wipe is hardcoded to 0 in https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L313.

- In consequence only the first address in addressesToWipe will ever be used to burn tokens in the for loop and the function does not work as expected.

- The function must now be called multiple times with a single address each to wipe a larger amount of addresses. This can be cumbersome and may cause a delay that prevents a quick wipe of multiple addresses.

## Tools Used

Manual review

## Recommended Mitigation Steps

Use i variable instead of hardcoded 0 when retrieving the address to wipe in the for loop.

---

# 5. INFINITY uses 1 << 255 instead of type(uint256).max

## Impact

LOW

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L47 uses "1 << 255" to define infinity. Why the current way of defining infinity is used is unclear since the maximum value in solidity (closest to infinity) would be "type(uint256).max" which has been established as a common practice to be used.

## Tools Used

Manual review

## Recommended Mitigation Steps

Use type(uint256).max to define infinity or add comment to explain why 1 << 255 is used.

---

# 6. Inconsistencies in code in MathUtil

## Impact

NON-CRITICAL

- \_cubicRoot is not using \_power3 in https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L24 "\_mulD18(\_mulD18(x, x), x)" is used which could be implemented using "\_power3(x)" instead.

- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L32 is using brackets where https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L36 is not using brackets for a similar expression.

## Tools Used

Manual review

## Recommended Mitigation Steps

- Use the \_power3 function from MathUtil.sol to achieve more consistency and remove code duplication.
- Use brackets consistently in similar expressions

---

# 7. Typos in multiple files

## Impact

NON-CRITICAL

Typos lead in the best case to irritation and in the worst case to wrong assumptions which may hurt the project.

### Equity.sol

- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L75 "see comment on the um" -> incomplete sentence
- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L197 "ensure helper unique" -> "ensure unique helper" or "ensure helper is unique"
- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L207 "helpes are necessary" -> " helpers are necessary"

### ERC20.sol

- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L24 "\* \*For a detailed " -> "\* For a detailed"

### Frankencoin.sol

- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L78 "Complex proposals should have application periods and applications fees above the minimum." -> "A complex proposal should have an application period and an applications fee above the minimum." (Singular for clarity!)

- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L188 "are only allowed to so for tokens amounts they previously" -> "are only allowed to burn amounts they previously" (please check if the suggestion is logically correct!)

- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L201 "the reserver requirement" -> "the reserve requirement"

### MintingHub.sol

- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L153 "the can split of smaller slices of the challenge" -> "the challenge can split into smaller slices of the challenge"

## Tools Used

Manual review

## Recommended Mitigation Steps

Fix the typos

---

# 8. Misleading function naming hints harmless function which actually have relevant implementation/side-effects

## Impact

NON-CRITICAL

- Several functions are named rather subtle starting with "notify" which e.g. implies some kind of message being sent which could be event emitting. But these function do significant things, e.g. assign state variables, revert under certain conditions and are used as checks in other functions.

- Since they are at least in parts called from another file one needs to dive into the function in the other contract to see that they actually do more than expected. This makes it even more likely that relevant things are overlooked (by the developers).

Affected functions:

- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L280 notifyLoss
- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L240 notifyRepaidInternal
- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L292 notifyChallengeStarted
- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L329 notifyChallengeSucceeded

## Tools Used

Manual review

## Recommended Mitigation Steps

Don't start these function names with "notify" and rename them to reflect their actual effects.

---

# 9. Issues with documentation

## Impact

NON-CRITICAL

- Generally the documentation does not follow NatSpec (https://docs.soliditylang.org/en/v0.8.19/natspec-format.html) consistently which is a flaw. E.g. it prevents tools to extract documentation properly which impacts security.

- Generally function parameters don't follow a consistent approach of prefixing them with underscore (\_). There is even cases where the same function has mixed usage of underscore for its parameters (see examples below).

- There are cases where documentation is not properly formatted (e.g. wrong indentation) which may hint that automated formatting for files is missing in the development environment (e.g. format on save).

### ERC20.sol

- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L177 Documentation out of sync with function parameter naming (to vs. recipient)

### ERC20PermitLight.sol

- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L11, https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L17 Indentation of comment is different

### MintingHub.sol

- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L54 does use underscore for "\_zchf" but not "factory" which is inconsistent.

### PositionFactory.sol

- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L14 does not use underscore for "initPeriod" which is inconsistent. All other parameters have underscore prefixes.

- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L36 comment is wrongly indented (2 whitespaces too much).

## Tools Used

Manual review

## Recommended Mitigation Steps

- Follow NatSpec (https://docs.soliditylang.org/en/v0.8.19/natspec-format.html) rigorously
- Follow consistent function parameter naming (use underscore prefixes for all of them)
- Adapt auto-formatting in your workflow

---

# 10. Not following suggested layout of contract files

## Impact

NON-CRITICAL

- https://docs.soliditylang.org/en/v0.8.19/style-guide.html suggests a widely adapted standard for contract file layout
- Not following this recommended contract layout files makes it harder for developers / auditors to gain overview

Examples:

- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L92 Errors are positioned at wrong place
- https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L299 Interface positioned at the wrong place

## Tools Used

Manual review

## Recommended Mitigation Steps

Follow the recommended contract layout

---

# 11. Referenced documentation file /doc/infiniteallowance.md is missing

## Impact

NON-CRITICAL

- In https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L130 the file /doc/infiniteallowance.md is referenced. But it is not in the repository (not in the code4rena Frankencoin repository and also not in the original repository https://github.com/Frankencoin-ZCHF/FrankenCoin/search?q=nfiniteallowance.md).

- In consequence a relevant part of documentation is missing for developers / auditors which impacts their work negatively through uncertainty.

## Tools Used

Manual review

## Recommended Mitigation Steps

- Either add this file to the repository or make it clear where this can be found (e.g. under https://docs.frankencoin.com/)
- If it is no more relevant, remove the comment that references /doc/infiniteallowance.md from the code

---

# 12. Insufficient code formatting

NON-CRITICAL

## Impact

- In some cases the code seems not formatted correctly. An example is the following:

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L85 "MIN_FEE && totalSupply()" (2 whitespaces between "MIN_FEE" and "&&").

## Tools Used

Manual review

## Recommended Mitigation Steps

- In this particular case remove the duplicate whitespace

- Generally use automated code formatting consistently across all files (e.g auto-format on save)

- If you use any CI tool (continuous integration) integrate a step that checks for proper code formatting after commit. It is rather recommended to not auto-format in CI since that may alter the code unexpectetly. Rather fail the CI pipeline if the code does not meet the expected formatting. Local pre-commit hooks may also be used to trigger any validations and prevent commits if code formatting is off.
