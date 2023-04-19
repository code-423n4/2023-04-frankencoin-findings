## Non-Critical Issues List
### Issue 
## [N-01] Undocumented assembly blocks ( 01 instances )
## [N-02] It's better to emit after all processing is done ( 07 instances)
## [N-03] Duplicate constant ( 01 instances )
## [N-04] Take advantage of Custom Error's return value property ( 09 instances )
## [N-05] Include the project name and development team information in the contract to increase the popularity and trust of users in the project
## [N-06] Project Upgrade and Stop Scenario should be
## [N-07] Use SMTChecker
## [N-08] Solidity compiler optimizations can be problematic
## [N-09] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier ( 04 instances )
### Total:  22 instances over 09  issues
## [N-01] Undocumented assembly blocks
### Description:
In `PositionFactory.sol` includes a function `createClone` with [an assembly block](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L39). Assembly is a low-level language that is harder to parse by readers. Consider including extensive inline documentation clearly explaining what every single assembly instruction does. This will make it easier for users to trust the code, for reviewers to verify it, and for developers to build on top of it or update it. Note that the use of assembly discards several important safety features of Solidity, which may render the code less safe and more error-prone. Hence, consider implementing thorough tests to cover all potential use cases of the `createClone` function to ensure it behaves as expected.
### Recommendation:
```
/* copied from https://github.com/optionality/clone-factory/blob/32782f82dfc5a00d103a7e61a17a5dedbd1e8e9d/contracts/CloneFactory.sol
 + Template Code for the create clone method:
+  function createClone(address target) internal returns (address result) {
+    bytes20 targetBytes = bytes20(target)${bytes == 20 ? "" : "<<" + ((20 - bytes) * 8)};
+    assembly {
+      let clone := mload(0x40)
+      mstore(clone, 0x${code.substring(0, 2*(cloner.labels.address + 1)).padEnd(64, '0')})
+      mstore(add(clone, 0x${(cloner.labels.address + 1).toString(16)}), targetBytes)
+      mstore(add(clone, 0x${(cloner.labels.address + bytes + 1).toString(16)}), 0x${code.substring(2*(cloner.labels.address + bytes + 1), 2*(cloner.labels.address+bytes+1) + 30).padEnd(64, '0')})
+      result := create(0, clone, 0x${(code.length / 2).toString(16)})
+    }
+  }
+ */
function createClone(address target) internal returns (address result) {
+        // convert address to bytes20 for assembly use
        bytes20 targetBytes = bytes20(target);
        assembly {
+            // allocate clone memory
            let clone := mload(0x40)
+            // store initial portion of the delegation contract code in bytes form
            mstore(clone, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
+            // store the provided address
            mstore(add(clone, 0x14), targetBytes)
+            // store the remaining delegation contract code
            mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
+            // create the actual delegate contract reference and return its address
            result := create(0, clone, 0x37)
        }
    }
```

## [N-02] It's better to emit after all processing is done ( 07 instances)
```
File: contracts/Equity.sol
249:        emit Trade(msg.sender, int(shares), amount, price());
280:        emit Trade(msg.sender, -int(shares), proceeds, price());
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L249
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L280

```
File: contracts/MintingHub.sol
146:        emit ChallengeStarted(msg.sender, address(position), _collateralAmount, pos);
176:        emit ChallengeStarted(challenge.challenger, address(challenge.position), challenge.size, _challengeNumber);
177:        emit ChallengeStarted(copy.challenger, address(copy.position), copy.size, pos);
212:            emit ChallengeAverted(address(challenge.position), _challengeNumber);
274:        emit ChallengeSucceeded(address(challenge.position), challenge.bid, _challengeNumber);
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L274

## [N-03] Duplicate constant
```
25: uint256 public constant MIN_FEE = 1000 * (10**18);
```
https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/Frankencoin.sol#L25
```
20: uint256 public constant OPENING_FEE = 1000 * 10**18;
```
https://github.com/code-423n4/2023-04-frankencoin/tree/main/contracts/MintingHub.sol#L20

## [N-04] Take advantage of Custom Error's return value property ( 09 instances )
An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the () sign, this kind of approach provides a serious advantage in debugging and examining the revert details of dapps such as tenderly.
```
File: contracts/Position.sol
77:        if(_coll < minimumCollateral) revert InsufficientCollateral();
81:        if (price > _price) revert InsufficientCollateral();
201:        if (block.timestamp >= challenge.end) revert TooLate();
202:        if (expectedSize != challenge.size) revert UnexpectedSize();
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L201-L202

```
File: contracts/Equity.sol
211:        if (_votes * 10000 < QUORUM * totalVotes()) revert NotQualified();
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L211

```
File: contracts/Frankencoin.sol
84:      if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
85:      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
86:      if (minters[_minter] != 0) revert AlreadyRegistered();
126:      if (!isMinter(msg.sender)) revert NotMinter();
153:      if (block.timestamp > minters[_minter]) revert TooLate();
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L153

## [N-05] Include the project name and development team information in the contract to increase the popularity and trust of users in the project
### Recommendation: Use form like FraxFinance project
https://github.com/FraxFinance/frxETH-public/blob/7f7731dbc93154131aba6e741b6116da05b25662/src/sfrxETH.sol#L4-L24

## [N-06] Project Upgrade and Stop Scenario should be
At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

## [N-07] Use SMTChecker
The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## [N-08] Solidity compiler optimizations can be problematic
```
96:          optimizer: {
97:              enabled: true,
98:              runs: 200
99:              },
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/hardhat.config.ts#L96-L99)
### Description: Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

Exploit Scenario A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

### Recommendation: Short term, measure the gas savings from optimizations and carefully weigh them against the possibility of an optimization-related bug. Long term, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## [N-09] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier
```
File: contracts/Position.sol
- 53:         require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values
+ 53:         require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values, 3 Days:  259200 (3 * 24 * 60 * 60 )
- 161:            restrictMinting(3 days);
+ 161:            restrictMinting(3 days); // 3 Days:  259200 (3 * 24 * 60 * 60 )
- 312:            restrictMinting(1 days);
+ 312:            restrictMinting(1 days); // 1 Days:  86400 (1 * 24 * 60 * 60 )
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L53
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L161
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L312
```
File: contracts/MintingHub.sol
- 64:            7 days, _expirationSeconds, _challengeSeconds, _mintingFeePPM, _liqPrice, _reservePPM);
+ 64:            7 days, _expirationSeconds, _challengeSeconds, _mintingFeePPM, _liqPrice, _reservePPM); // 7 Days:  604800 (7 * 24 * 60 * 60 )
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L64