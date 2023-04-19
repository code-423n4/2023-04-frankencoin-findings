### [QA-01] Launching a challenge can be used as a method of blocking a position

### [QA-02] Tokens can be sent to voters to decrease their voting power and gain control over the voting mechanism

### [QA-03] Partial challenging

Depending on exact position parameters a challenger could to some extent make it difficult for bidders to place a bid by mangling with splitting their challenge into smaller ones.

Did you consider a design where instead of having to bid on the whole challenged amount and having to split it in case its too big, bidders could simply place
a partial bid?

Once the challenge is successfully complete, the top unfulfilled part of the bidders could simply receive the adequate amount of collateral from the position owner.

Doing so could reduce work needed to keep track of all open parts of the challenge, possibly making the protocol easier and more gas efficient for the end user.

### [QA-04] Typos

```
- event PostPonedReturn(address collateral, address indexed beneficiary, uint256 amount);
+ event PostponedReturn(address collateral, address indexed beneficiary, uint256 amount);
```

```
- * have the liquidity available to bid a sufficient amount. With this function, the can split of smaller slices of
+ * the challenge and avert it piece by piece.

- * have the liquidity available to bid a sufficient amount. With this function, they can split
+ * the challenge to smaller slices and avert it piece by piece.
```

```
- * Here, we are also save, as 68 Bits would imply more than a trillion outstanding shares. In fact, when
+ * Here, we are also safe, as 68 Bits would imply more than a trillion outstanding shares. In fact, when
```

```
- If the effective interest at which new positions can be opened is at 5% and the reserve is below the targed of one 
+ If the effective interest at which new positions can be opened is at 5% and the reserve is below the target of one 
```

### [QA-05] Unspecific compiler version pragma

Avoid floating pragmas for non-library contracts.  
It is recommended to pin to a concrete compiler version.  

[contracts/Equity.sol#L4](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L4)  
```
pragma solidity >=0.8.0 <0.9.0;
```
[contracts/ERC20.sol#L12](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L12)  
[contracts/ERC20PermitLight.sol#L5](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L5)  
[contracts/Frankencoin.sol#L2](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L2)  
[contracts/MathUtil.sol#L3](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L3)  
[contracts/MintingHub.sol#L2](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L2)  
[contracts/Ownable.sol#L9](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L9)  
[contracts/Position.sol#L2](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L2)  
[contracts/PositionFactory.sol#L2](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L2)  
[contracts/StablecoinBridge.sol#L2](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L2)  

### [QA-06] Code formatting

[contracts/MathUtil.sol#L25](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L25)  
```
- x = _mulD18(x, _divD18( (powX3 + 2 * _v) , (2 * powX3 + _v)));
+ x = _mulD18(x, _divD18((powX3 + 2 * _v), (2 * powX3 + _v)));
```

### [QA-07] Lines too long

Keep line width to max 120 characters for better readability where possible.  

[contracts/ERC20PermitLight.sol#L40](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L40)  
[contracts/Equity.sol#L117](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L117)  
[contracts/Equity.sol#L161](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L161)  
[contracts/Equity.sol#L268](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L268)  
[contracts/Frankencoin.sol#L78](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L78)  
[contracts/Frankencoin.sol#L79](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L79)  
[contracts/Frankencoin.sol#L80](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L80)  
[contracts/Frankencoin.sol#L83](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L83)  
[contracts/Frankencoin.sol#L115](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L115)  
[contracts/Frankencoin.sol#L169](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L169)  
[contracts/Frankencoin.sol#L185](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L185)  
[contracts/Frankencoin.sol#L188](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L188)  
[contracts/Frankencoin.sol#L190](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L190)  
[contracts/Frankencoin.sol#L191](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L191)  
[contracts/Frankencoin.sol#L192](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L192)  
[contracts/Frankencoin.sol#L201](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L201)  
[contracts/Frankencoin.sol#L217](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L217)  
[contracts/Frankencoin.sol#L219](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L219)  
[contracts/Frankencoin.sol#L220](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L220)  
[contracts/Frankencoin.sol#L221](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L221)  
[contracts/Frankencoin.sol#L223](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L223)  
[contracts/Frankencoin.sol#L232](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L232)  
[contracts/Frankencoin.sol#L235](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L235)  
[contracts/Frankencoin.sol#L238](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L238)  
[contracts/Frankencoin.sol#L243](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L243)  
[contracts/Frankencoin.sol#L244](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L244)  
[contracts/Frankencoin.sol#L246](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L246)  
[contracts/Frankencoin.sol#L248](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L248)  
[contracts/Frankencoin.sol#L251](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L251)  
[contracts/Frankencoin.sol#L254](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L254)  
[contracts/MintingHub.sol#L121](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L121)  
[contracts/MintingHub.sol#L122](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L122)  
[contracts/MintingHub.sol#L124](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L124)  
[contracts/MintingHub.sol#L140](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L140)  
[contracts/MintingHub.sol#L144](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L144)  
[contracts/MintingHub.sol#L247](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L247)  
[contracts/MintingHub.sol#L250](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L250)  
[contracts/MintingHub.sol#L256](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L256)  
[contracts/MintingHub.sol#L258](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L258)  
[contracts/MintingHub.sol#L260](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L260)  
[contracts/MintingHub.sol#L289](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L289)  
[contracts/MintingHub.sol#L294](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L294)  
[contracts/Position.sol#L21](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L21)  
[contracts/Position.sol#L76](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L76)  
[contracts/Position.sol#L221](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L221)  
[contracts/Position.sol#L227](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L227)  
[contracts/Position.sol#L229](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L229)  
[contracts/Position.sol#L230](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L230)  
[contracts/Position.sol#L231](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L231)  
[contracts/Position.sol#L232](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L232)  
[contracts/Position.sol#L335](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L335)  
[contracts/Position.sol#L337](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L337)  
[contracts/Position.sol#L355](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L355)  
[contracts/PositionFactory.sol#L36](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L36)  
[contracts/ERC20.sol#L53](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L53)  

### [QA-08] Use a more recent version of solidity

[contracts/Frankencoin.sol#L2](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L2)  
```
pragma solidity ^0.8.0;
```
[contracts/ERC20PermitLight.sol#L5](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L5)  
[contracts/Equity.sol#L4](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L4)  
[contracts/MathUtil.sol#L3](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L3)  
[contracts/MintingHub.sol#L2](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L2)  
[contracts/Ownable.sol#L9](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L9)  
[contracts/Position.sol#L2](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L2)  
[contracts/PositionFactory.sol#L2](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L2)  
[contracts/StablecoinBridge.sol#L2](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L2)  
[contracts/ERC20.sol#L12](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L12)  
