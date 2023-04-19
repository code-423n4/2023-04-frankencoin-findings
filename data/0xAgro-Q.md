# QA Report
## Finding Summary

[**Low Severity**](#Low-Severity)
1. [**Unchecked Cast May Overflow**](#1-Unchecked-Cast-May-Overflow)

[**Non-Critical**](#Non-Critical)
1. [**Use of Exponentiation Over Scientific Notation**](#1-Use-of-Exponentiation-Over-Scientific-Notation)
2. [**Explicit Variable Not Used**](#2-Explicit-Variable-Not-Used)
3. [**Inconsistent Comment Initial Spacing**](#3-Inconsistent-Comment-Initial-Spacing)
4. [**Inconsistent Named Returns**](#4-Inconsistent-Named-Returns)
5. [**Spelling Mistakes**](#5-Spelling-Mistakes)
6. [**ERC20 Token Recovery**](#6-ERC20-Token-Recovery)
7. [**Multi-Line Comment Inconsistency**](#7-Multi-Line-Comment-Inconsistency)
8. [**Trailing Comment Spacing Inconsistency**](#8-Trailing-Comment-Spacing-Inconsistency)
9. [**Extra Newline**](#9-Extra-Newline)
10. [**Unconventional Underscore Notation**](#10-Unconventional-Underscore-Notation)
11. [**Inconsistent Underscore Notation**](#11-Inconsistent-Underscore-Notation)
12. [**Lack Of Code Seperation**](#12-Lack-Of-Code-Seperation)
13. [**Lack Of Brace Spacing**](#13-Lack-Of-Brace-Spacing)
14. [**Unnecessary Inline Mapping Comments**](#14-Unnecessary-Inline-Mapping-Comments)
15. [**Unnecessary Inline Param Comments**](#15-Unnecessary-Inline-Param-Comments)
16. [**Extra Spacing**](#16-Extra-Spacing)
17. [**Inconsistent NatSpec Spacing**](#17-Inconsistent-NatSpec-Spacing)
18. [**Inconsistant Header Spacing**](#18-Inconsistant-Header-Spacing)
19. [**Unnecessary Headers**](#19-Unnecessary-Headers)
20. [**Newline Between Pragma and License**](#20-Newline-Between-Pragma-and-License)
21. [**Empty Comments**](#21-Empty-Comments)

[**Style Guide Violations**](#Style-Guide-Violations)
1. [**Whitespace in Expressions**](#1-Whitespace-in-Expressions)
2. [**Mappings**](#2-Mappings)
3. [**Lack Of Whitespace In Operation**](#3-Lack-Of-Whitespace-In-Operation)
4. [**Function Declaration Modifier Order**](#4-Function-Declaration-Modifier-Order)
5. [**Order of Layout**](#5-Order-of-Layout)

[**Automated Report Misses**](#Automated-Report-Misses)
1. [**Error Messages Are Missing From Require Statements**](#1-Error-Messages-Are-Missing-From-Require-Statements)
2. [**Underscore Notation Not Used or Not Used Consistently**](#2-Underscore-Notation-Not-Used-or-Not-Used-Consistently)

# Low Severity

## 1. Unchecked Cast May Overflow

As of Solidity 0.8 overflows are handled automatically; however, not for casting. For example `uint32(4294967300)` will result in `4` without reversion. Consider using OpenZepplin's SafeCast library. Even if it seems as though a value cannot overflow, it is best to be safe.

*/contracts/Position.sol*

```solidity
187:	return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));
```

*/contracts/Equity.sol*

```solidity
146:	totalVotesAtAnchor = uint192(totalVotes() - roundingLoss - lostVotes);
161:	voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance);
173:	return uint64(block.number << BLOCK_TIME_RESOLUTION_BITS);
```

# Non-Critical

## 1. Use of Exponentiation Over Scientific Notation

For better style and less computation consider replacing any power of 10 exponentiation (`10**3`) with its equivalent scientific notation (`1e3`).

*/contracts/MintingHub.sol*

```solidity
20:	uint256 public constant OPENING_FEE = 1000 * 10**18;
```

*/contracts/Frankencoin.sol*

```solidity
25:	uint256 public constant MIN_FEE = 1000 * (10**18);
```

## 2. Explicit Variable Not Used

As described in the [Solidity documentation](https://docs.soliditylang.org/en/v0.4.21/types.html#integers): 
> "`uint` and `int` are aliases for `uint256` and `int256`, respectively". 

There are moments in the codebase where `uint` / `int` is used instead of the explicit `uint256` / `int256`. It is best to be explicit with variable names to lessen confusion. Consider replacing instances of `uint` / `int` with `uint256` / `int256`.

*/contracts/Equity.sol*

```solidity
91:	event Trade(address who, int amount, uint totPrice, uint newprice);
249:	emit Trade(msg.sender, int(shares), amount, price());
```

## 3. Inconsistent Comment Initial Spacing

Some comments have an initial space after `//` or `///` while others do not. It is best for code clearity to keep a consistent style.

1. The following contracts only have initial space comments (EX. `// foo`): [Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol), [MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol), [Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol), [Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol), [ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol), [StablecoinBridge.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol), [PositionFactory.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol), and [Ownable.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol).
2. No contracts have only no initial space comments (EX. `//foo`).
3. The following contracts have both: [ERC20PermitLight.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol), and [MathUtil.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol).

## 4. Inconsistent Named Returns

Some functions use named returns and others do not. It is best for code clearity to keep a consistent style.

1. No contracts only have named returns (EX. `returns(uint256 foo)`).
2. The following contracts only have non-named returns (EX. `returns(uint256)`): [Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol), [MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol), [Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol), [Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol), [ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol), [ERC20PermitLight.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol), [StablecoinBridge.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol), and [MathUtil.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol).
3. The following contracts have both: [PositionFactory.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol).

## 5. Spelling Mistakes

There are some spelling mistakes throughout the codebase. Consider fixing all spelling mistakes.

*contracts/Position.sol*

* The word `criterion` is misspelled as [`creterion`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L358).

*contracts/Equity.sol*

* The word `proportional` is misspelled as [`proporational`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L34).
* The word `inflation` is misspelled as [`inflaction`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L73).
* The word `helps` is misspelled as [`helpes`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L207).

*contracts/Frankencoin.sol*

* The word `arbitrary` is misspelled as [`arbitraty`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol).

## 6. ERC20 Token Recovery

Consider adding a recovery function for Tokens that are accidently sent to the core contracts of the protocol. 

## 7. Multi-Line Comment Inconsistency

There are two differing styles of multi-line comment in the codebase:

**Style 1** [ref](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L11-L13)
```Solidity
11: /**
12:  * A collateralized minting position.
13:  */
```

**Style 2** [ref](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L47-L49)
```Solidity
47: /**
48: * See MintingHub.openPosition
49: */
```

Notice how **style 2** has no leading space. Consider sticking to a single style. 

## 8. Trailing Comment Spacing Inconsistency

[This](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L11) line has no trailing comment space (between end of line and `//`). All other comments ([ex](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L13)) have a trailing comment space. 

**Example 1**
```Solidity
11: uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01
```

**Example 2**
```Solidity
13: IERC20 public immutable chf; // the source stablecoin
```

Consider adding a space.

## 9. Extra Newline

There is [an extra newline](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L1) only found in `Equity.sol` that should be removed.

## 10. Unconventional Underscore Notation

Normally, [underscore notation](https://docs.soliditylang.org/en/latest/types.html#rational-and-integer-literals) for the number `1000000` is written as `1_000_000`. There are many times in the codebase where `1000000` is written as `1000_000`.

*/contracts/Position.sol*
```Solidity
122: return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
124: return totalMint * (1000_000 - reserveContribution) / 1000_000;
```

*/contracts/MintingHub.sol*
```Solidity
265: uint256 reward = (volume * CHALLENGER_REWARD) / 1000_000;
```

*/contracts/Frankencoin.sol*
```Solidity
166: uint256 usableMint = (_amount * (1000_000 - _feesPPM - _reservePPM)) / 1000_000; // rounding down is fine
```

Consider changing all occurances of `1000_000` to `1_000_000`.

## 11. Inconsistent Underscore Notation

The number `1000000` can be seen written 2 different ways, `1000000` ([ex](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L166)) and `1000_000` ([ex](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L205)). Consider using one style.

## 12. Lack Of Code Seperation

There is a general lack of code seperation in the codebase. For example [this](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L51-L67) block has no spacing between lines or even worse [this](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L253-L275) block. A good example of spacing can be seen [here](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L180-L186). Consider adding newlines between `require` statements and other code for example.

## 13. Lack Of Brace Spacing

There are many instances of non-spaced function / conditional braces. Consider adding a space between the function / conditional closing parenthesis and brace.

*/contracts/Position.sol*
```solidity
184: if (time >= exp){
169: function collateralBalance() internal view returns (uint256){
```

*/contracts/Frankencoin.sol*
```solidity
104: if (explicit > 0){
207: if (currentReserve < minterReserve()){
235: function calculateFreedAmount(uint256 amountExcludingReserve /* 41 */, uint32 reservePPM /* 20% */) public view returns (uint256){
293: function isMinter(address _minter) override public view returns (bool){
300: function isPosition(address _position) override public view returns (address){
```

*/contracts/Equity.sol*
```solidity
172: function anchorTime() internal view returns (uint64){
```

*/contracts/Position.sol*
```solidity
137: if (newCollateral > colbal){
141: if (newMinted < minted){
145: if (newCollateral < colbal){
149: if (newMinted > minted){
```

## 14. Unnecessary Inline Mapping Comments

There is mapping with an internal comment `/** col */`. Normally an inline comment would not be an issue; however, it is already described 2 lines above. Consider removing the internal comment.

*/contracts/MintingHub.sol*

```solidity
35:   * It maps collateral => beneficiary => amount.
37: mapping (address /** col */ => mapping (address => uint256)) public pendingReturns;
```

## 15. Unnecessary Inline Param Comments

There is an inline comment describing parameters seen [here](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L235). Parameters should be described in a [NatSpec](https://docs.soliditylang.org/en/v0.8.17/natspec-format.html) `@param` tagged comment instead of an inline comment.

*/contracts/Frankencoin.sol*
```solidity
235: function calculateFreedAmount(uint256 amountExcludingReserve /* 41 */, uint32 reservePPM /* 20% */) public view returns (uint256){
```

## 16. Extra Spacing 

There are a couple extra spaces found in the codebase, namely, [here](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L68) (see end) and [here](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L36) (see start). 

## 17. Inconsistent NatSpec Spacing

The NatSpec `param` comments are alined seen in [this](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L136-L137) example; however not [here](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L76-L83) (see [L76](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L76)). 

## 18. Inconsistant Header Spacing

The headers [here](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L11) and [here](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L17) differ in initial line spacing (3 spaces vs 2). Consider removing a space or adding a space respectively.

```solidity
11:   /*//////////////////////////////////////////////////////////////
12:                            EIP-2612 STORAGE
13:    //////////////////////////////////////////////////////////////*/
```

```solidity
17:  /*//////////////////////////////////////////////////////////////
18:                             EIP-2612 LOGIC
19:    //////////////////////////////////////////////////////////////*/
```

## 19. Unnecessary Headers

There are generic headers seen [here](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L11-L13) and [here](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L17-L19). Generic headers help with readability, but add unnecessary bulk - as long as the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#introduction) is followed. Specifically, the two key elements that should be followed from the Style Guide are [Order of Layout](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout) and [Order of Functions](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions). Consider removing headers.

## 20. Newline Between Pragma and License

[This](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L2) contract and [this](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L3) contract have a newline between the pragma and license. Although it does not void the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-lengthhttps://docs.soliditylang.org/en/v0.8.17/style-guide.html), all example contracts in the Style Guide do not have a space between the license statement and pragma ([ex](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#blank-lines)). This format can also be seen throughout the Solidy Documentation ([ex](https://docs.soliditylang.org/en/v0.8.17/natspec-format.html)). Consider removing the needless space.

## 21. Empty Comments

Empty comments that should be considered for removal can be found [here](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L2) and [here](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L4). 

# Style Guide Violations

## 1. Whitespace in Expressions

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#whitespace-in-expressions) recommends to 
> *"[a]void extraneous whitespace [i]mmediately inside parenthesis, brackets or braces, with the exception of single line function declarations"*. 

The Style Guide also recommends against spaces before semicolons. Consider removing spaces before semicolons.

*/contracts/Frankencoin.sol*

```solidity
235:	function calculateFreedAmount(uint256 amountExcludingReserve , uint32 reservePPM ) public view returns (uint256){
```

*/contracts/MathUtil.sol*

```solidity
25:	x = _mulD18(x, _divD18( (powX3 + 2 * _v) , (2 * powX3 + _v)));
27:	} while ( cond );
36:	return (_a * ONE_DEC18) / _b ;
```

## 2. Mappings

Mapping whitespace not in accordance with the [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#mappings).

*/contracts/MintingHub.sol*

```solidity
37:	mapping (address  => mapping (address => uint256)) public pendingReturns;
```

*/contracts/Equity.sol*

```solidity
83:	mapping (address => address) public delegates;
88:	mapping (address => uint64) private voteAnchor;
```

*/contracts/Frankencoin.sol*

```solidity
45:	mapping (address => uint256) public minters;
50:	mapping (address => address) public positions;
```

*/contracts/ERC20.sol*

```solidity
43:	mapping (address => uint256) private _balances;
45:	mapping (address => mapping (address => uint256)) private _allowances;
```

## 3. Lack Of Whitespace In Operation

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#other-recommendations) recommends operations have a single whitespace before and after.

*/contracts/Position.sol*

```solidity
98:	uint256 reduction = (limit - minted - _minimum)/2;
```

*/contracts/Equity.sol*

```solidity
59:	uint256 public constant MIN_HOLDING_DURATION = 90*7200 << BLOCK_TIME_RESOLUTION_BITS;
192:	for (uint i=0; i<helpers.length; i++){
196:	for (uint j=i+1; j<helpers.length; j++){
312:	for (uint256 i = 0; i<addressesToWipe.length; i++){
```

## 4. Function Declaration Modifier Order

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#function-declaration) suggests the following modifer order:  Visibility, Mutability, Virtual, Override, Custom modifiers.

*contracts/Equity.sol*

* `name` ('override', 'external', 'pure'): pure (Mutability) positioned after override (Override).
* `symbol` ('override', 'external', 'pure'): pure (Mutability) positioned after override (Override).
* `_beforeTokenTransfer` ('override', 'internal'): internal (Visibility) positioned after override (Override).
* `checkQualified` ('public', 'override', 'view'): view (Mutability) positioned after override (Override).

*contracts/Frankencoin.sol*

* `name` ('override', 'external', 'pure'): pure (Mutability) positioned after override (Override).
* `symbol` ('override', 'external', 'pure'): pure (Mutability) positioned after override (Override).
* `suggestMinter` ('override', 'external'): external (Visibility) positioned after override (Override).
* `registerPosition` ('override', 'external'): external (Visibility) positioned after override (Override).
* `denyMinter` ('override', 'external'): external (Visibility) positioned after override (Override).
* `mint` ('override', 'external', 'Custom'): CUSTOM (Custom Modifiers) positioned after override (Override).
* `burn` ('override', 'external', 'Custom'): CUSTOM (Custom Modifiers) positioned after override (Override).
* `notifyLoss` ('override', 'external', 'Custom'): CUSTOM (Custom Modifiers) positioned after override (Override).
* `isMinter` ('override', 'public', 'view'): view (Mutability) positioned after override (Override).
* `isPosition` ('override', 'public', 'view'): view (Mutability) positioned after override (Override).

*contracts/ERC20.sol*

* `_beforeTokenTransfer` ('virtual', 'internal'): internal (Visibility) positioned after virtual (Virtual).

## 5. Order of Layout

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout) suggests the following contract layout order: Type declarations, State variables, Events, Modifiers, Functions.

The following contracts are not compliant (examples are only to prove the layout are out of order NOT a full description): 

* [Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol): Modifiers are positioned after Functions.
* [MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol): Modifiers are positioned after Functions.
* [Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol): Modifiers are positioned after Functions.
* [Ownable.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol): Modifiers are positioned after Functions.

## 6. Function Declaration Style Long

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#function-declaration) states that: 
> "[f]or long function declarations, it is recommended to drop each argument onto its own line at the same indentation level as the function body. The closing parenthesis and opening bracket should be placed on their own line as well at the same indentation level as the function declaration".

The following functions are in violation:

*/contracts/Position.sol*

* [`constructor`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L50-L52).

*/contracts/MintingHub.sol*

* [`openPosition`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L60-L62).

*/contracts/PositionFactory.sol*

* [`createNewPosition`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L13-L15).


[This](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L21-L29) is a good example from the codebase that is not in violation.

# Automated Report Misses

## 1. Error Messages Are Missing From Require Statements

*This issue adds missed items to the automated audit report found [here](https://gist.github.com/CodingNameKiki/36f3bfb214907d68fdf3a43cb0cb8ae3#nc22-require--revert-statements-should-have-descriptive-reason-strings)*

It is good practice to return a message on failure. Error messages help with debugging.

*/contracts/Position.sol*

```solidity
53:	require(initPeriod >= 3 days);
```

## 2. Underscore Notation Not Used or Not Used Consistently

*This issue adds missed items to the automated audit report found [here](https://gist.github.com/CodingNameKiki/36f3bfb214907d68fdf3a43cb0cb8ae3#nc25-use-underscores-for-number-literals)*

Consider using underscore notation to help with contract readability (Ex. `23453` -> `23_453`).

*/contracts/MintingHub.sol*

```solidity
20:	uint256 public constant OPENING_FEE = 1000 * 10**18;
26:	uint32 public constant CHALLENGER_REWARD = 20000;
189:	return (challenge.bid * 1005) / 1000;
265:	uint256 reward = (volume * CHALLENGER_REWARD) / 1000_000;
```

*/contracts/Equity.sol*

```solidity
41:	uint256 private constant MINIMUM_EQUITY = 1000 * ONE_DEC18;
59:	uint256 public constant MIN_HOLDING_DURATION = 90*7200 << BLOCK_TIME_RESOLUTION_BITS;
247:	uint256 shares = equity <= amount ? 1000 * ONE_DEC18 : calculateSharesInternal(equity - amount, amount);
268:	uint256 newTotalShares = totalShares < 1000 * ONE_DEC18 ? 1000 * ONE_DEC18 : _mulD18(totalShares, _cubicRoot(_divD18(capitalBefore + investment, capitalBefore)));
```

*/contracts/Frankencoin.sol*

```solidity
25:	uint256 public constant MIN_FEE = 1000 * (10**18);
166:	uint256 usableMint = (_amount * (1000_000 - _feesPPM - _reservePPM)) / 1000_000;
```