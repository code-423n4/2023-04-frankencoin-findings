### Low Risk Issues List

| Number | Issue                   | Instances |
| :----: | :---------------------- | :-------: |
| [L-01] | Unsafe casting of uints |     2     |

#### [L-01] Unsafe casting of uints

Downcasting from uint256 in Solidity does not revert on overflow. This can result in undesired exploitation or bugs, since developers usually assume that overflows raise errors. [OpenZeppelin's SafeCast](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast) restores this intuition by reverting the transaction when such an operation overflows. Using this library instead of the unchecked operations eliminates an entire class of bugs, so it’s recommended to always use it.

For example:

```solidity
// Before
return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));

// After
return toUint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));
```

1. In [Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol)

   ```solidity
   File: contracts/Position.sol

   187:    return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));
   ```

1. In [Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol)

   ```solidity
   File: contracts/Equity.sol

   146:    totalVotesAtAnchor = uint192(totalVotes() - roundingLoss - lostVotes);

   161:    voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance);
   ```

### Non Critical Issues

| Number  | Issue                               | Instances |
| :-----: | :---------------------------------- | :-------: |
| [NC-01] | Inconsistent code formatting        |   many    |
| [NC-02] | Use the complete name of data types |     5     |

#### [NC-01] Inconsistent code formatting

This code base exhibits inconsistencies that may be interpreted by a reader as a lack of attention to detail which could reduce confidence and trust in the protocol. The lack of consistent formatting also makes reading and maintaining the code more difficult.

To increase readability and maintainability, and to increase confidence in the protocol and the team behind it, the Solidity source code should use consistent formatting throughout.

Mitigation:

1. Follow the [Solidity Style Guide (SSG)](https://docs.soliditylang.org/en/latest/style-guide.htm)
2. Use `forge fmt`, [prettier-solidity](https://github.com/prettier-solidity/prettier-plugin-solidity), or an equivalent solution to automatically format Solidity code.

There are many examples of inconsistent bracing styles, inconsistent horizontal whitespace, and generally not following the Solidity style guide. Below are some examples.

1. In [Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol)

   ```solidity
   File: contracts/Position.sol

   // No spaces surrounding division operator on L98, but spaces are present on L122.
   // SSG: "Surround operators with a single space on either side."
   98:    uint256 reduction = (limit - minted - _minimum)/2;
   122:   return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;

   // No space before opening curly brace on L121, but space is present on L160.
   // SSG: "The opening brace should be preceded by a single space."
   121:   if (afterFees){
   160:   if (newPrice > price) {
   ```

1. In [Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol)

   ```solidity
   File: contracts/Equity.sol

   // No spaces surrounding operators in for loop on L192, but are partially in place on L312.
   // SSG: "Surround operators with a single space on either side."
   192:    for (uint i=0; i<helpers.length; i++){
   312:    for (uint256 i = 0; i<addressesToWipe.length; i++){

   // No space before opening curly brace on L172, but space is present on L179.
   // SSG: "The opening brace should be preceded by a single space."
   172:   function anchorTime() internal view returns (uint64){
   179:   function votes(address holder) public view returns (uint256) {
   ```

1. In [Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol)

   ```solidity
   File: contracts/Frankencoin.sol

   // Inconsistent indentation: The other contracts are using a four-space indent. This contract file uses a mix of two and three space indentation.

   // Three space indentation.
   64:    function name() override external pure returns (string memory){
   65:       return "Frankencoin";
   66:    }

   // Two space indentation.
   141:    if (balance <= minReserve){
   142:      return 0;
   143:    } else {
   144:      return balance - minReserve;
   145:    }
   ```

1. In [ERC20PermitLight.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol)

   ```solidity
   File: contracts/ERC20PermitLight.sol

   // Space after start of single line comment.
   40:    // keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),

   // No space after start of single line comment.
   65:    //keccak256("EIP712Domain(uint256 chainId,address verifyingContract)");
   ```

#### [NC-02] Use the complete name of data types

Subtle bugs can occur when just `uint` or `int` is used. For example, when hashing the signature of a function and use `uint` instead of `uint256` for a parameter type.

Remediation recommendations:

1. Replace `uint` with `uint256`. Replace `int` with `int256`.
2. Use `forge fmt`, which will automatically change `uint` to `uint256` and `int` to `int256`.

```solidity
File: contracts/Equity.sol

91:    event Trade(address who, int amount, uint totPrice, uint newprice);

192:   for (uint i=0; i<helpers.length; i++){

196:   for (uint j=i+1; j<helpers.length; j++){
```
