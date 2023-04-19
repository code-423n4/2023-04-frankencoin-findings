| Non-Critical Issues List |                                                                                                                   |         |
|------------------------|-------------------------------------------------------------------------------------------------------------------|---------|
| Number                 | Issues Details                                                                                              | Context |
| [N-01]                 | Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)                                                              | 3      |
| [N-02]                 | Large multiples of ten should use scientific notation (e.g. 1e6) rather than decimal literals (e.g. 1000000), for readability | 4       |
| [N-03]                 | Avoid the use of sensitive terms                                                                                     | 2        |
| [N-04]                 | Not Completely Using OpenZeppelin Contracts                                                                                          | 1       |
| [N-05]                 | Contracts should have full test coverage                                                                                         |        |
| [N-06]                 | According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword                                                                                      | 1       |
| [N-07]                 | Function writing that does not comply with the Solidity Style Guide|All Contracts|

Total 7 issues

###### [N-1] Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)

While the compiler knows to optimize away the exponentiation, it’s still better coding practice to use idioms that do not require compiler optimization, if they exist.
```
contracts\MintingHub.sol:
20:    uint256 public constant OPENING_FEE = 1000 * 10**18;
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L20>
```
contracts\Equity.sol:
253:   require(totalSupply() < 2**128, "total supply exceeded");
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L253>
```
contracts\Math.util:
10    uint256 internal constant ONE_DEC18 = 10**18; 
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L10>

###### [N-2] Large multiples of ten should use scientific notation (e.g. 1e6) rather than decimal literals (e.g. 1000000), for readability

```
contracts\Position.sol:
122:   return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L122>
```
contracts\Position.sol:
124:   return totalMint * (1000_000 - reserveContribution) / 1000_000;
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L124>
```
contracts\MintingHub.sol:
256:   uint256 reward = (volume * CHALLENGER_REWARD) / 1000_000;
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L265>
```
contracts\Math.util:
11     uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L11>

###### [N-3] Avoid the use of sensitive terms

Use alternative variants, e.g. allowlist/denylist instead of whitelist/blacklist

https://www.zdnet.com/article/mysql-drops-master-slave-and-blacklist-whitelist-terminology/

blacklist-blocklist
```
contracts\MintingHub.sol
247     * In case that the collateral cannot be transfered back to the challenger (i.e. because the collateral token has a blacklist and the 
        * challenger is on it), it is possible to postpone the return of the collateral.
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L247>
```
contracts\MintingHub.sol
289     // Postponing helps in case the challenger was blacklisted on the collateral token or otherwise cannot receive it at the moment.
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L289>


###### [N-4] Not Completely Using OpenZeppelin Contracts

OpenZeppelin maintains a library of standard, audited, community-reviewed, and battle-tested smart contracts. Instead of always importing these contracts, the protocol project re-implements them in some cases. This increases the amount of code that the protocol team will have to maintain and miss all the improvements and bug fixes that the OpenZeppelin team is constantly implementing with the help of the community.

Consider importing the OpenZeppelin contracts instead of re-implementing or copying them. These contracts can be extended to add the extra functionalities required if need be.

Here is one of the instances entailed:
```
contracts\ReentrancyGuard.sol
6    /**
     * @title ReentrancyGuard
     * @notice This contract protects against reentrancy attacks.
     *         It is adjusted from OpenZeppelin.
     */
```
<https://github.com/code-423n4/2022-11-looksrare/blob/main/contracts/ReentrancyGuard.sol#L6-L10>

###### [N-5] Contracts should have full test coverage

While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit

###### [N-6] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword
```
contracts\ERC20PermitLight.sol
15:    mapping(address => uint256) public nonces;
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L15>

###### [N-7] Function writing that does not comply with the Solidity Style Guide
Context: All Contracts
Description: Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.
<https://docs.soliditylang.org/en/v0.8.17/style-guide.html>
