# [L-01] Oracle failure check
## Lines of code
[Position.sol#L316](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L316)
## Vulnerability details
### Proof of concept
````solidity
contracts/utilities/poolsUtility/Position.sol

316:    uint256 _price = oracle.getUnderlyingPrice(CToken(_bathToken));
````
## Tools Used
Manual Review
## Recommended mitigation steps
It is good to check the returned value from oracle is sane (i.e. if it is zero) and prepare for a fallback.

# [L-02] Call to `Position.openPosition()` can reenter
## Lines of code
[Position.sol#L125-L206](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L125-L206)
## Vulnerability details
### Impact
The openPosition function in Position does several external calls to open a position. This call is unsafe, as the recipient can be malicious and do a reentrancy call.
## Tools Used
Manual Review
## Recommended mitigation steps
Consider adding a `nonReentrant` modifier to the method.

# [L-03] Use `2StepSetOwner` instead of `setOwner`
## Lines of code
[RubiconMarket.sol#L25-L28](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L25-L28)
## Vulnerability details
### Proof of concept
````solidity
contracts/RubiconMarket.sol

25      function setOwner(address owner_) external auth {
26          owner = owner_;
27          emit LogSetOwner(owner);
28:     }
````
## Tools Used
Manual Review
## Recommended mitigation steps
Use a 2 structure which is safer.

# [I-01] Using `SafeMath` when compiler is `^0.8.0`
## Lines of code
[BathBuddy.sol#L2](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/periphery/BathBuddy.sol#L2), [BathBuddy.sol#L6](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/periphery/BathBuddy.sol#L6)
## Vulnerability details
There is no need to use `SafeMath` when compiler is `^0.8.0` because it has built-in under/overflow checks.