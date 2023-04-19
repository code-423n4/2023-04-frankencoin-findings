| Non-Critical Issues List |                                                                                                                   |         |
|------------------------|-------------------------------------------------------------------------------------------------------------------|---------|
| Number                 | Issues Details                                                                                              | Context |
| [N-01]                 | Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)                                                              | 3      |
| [N-02]                 | Large multiples of ten should use scientific notation (e.g. 1e6) rather than decimal literals (e.g. 1000000), for readability | 4       |
| [N-03]                 | Avoid the use of sensitive terms                                                                                     | 2        |
| [N-04]                 | According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword                                                                                      | 1       |
| [N-05]                 | Function ordering does not follow the Solidity style guide|All Contracts|

Total 5 issues

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

###### [N-4] According to the syntax rules, use => mapping ( instead of => mapping( using spaces as keyword
```
contracts\ERC20PermitLight.sol
15:    mapping(address => uint256) public nonces;
```
<https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L15>

###### [N-5] Function ordering does not follow the Solidity style guide
Context: All Contracts
According to the Solidity style guide, functions should be laid out in the following order :constructor(), receive(), fallback(), external, public, internal, private, but the cases below do not follow this pattern.
<https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions>