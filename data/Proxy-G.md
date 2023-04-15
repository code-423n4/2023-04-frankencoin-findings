## Gas Optimizations

### Summary

|        | Issue                                                 | Instances | Total Gas Saved |
| ------ | :---------------------------------------------------- | :-------: | :-------------: |
| [G-01] | Cheaper Comparison                                    |    17     |       51        |
| [G-02] | Consider using assembly for math (add, sub, mul, div) |    43     |      ~1800      |
| [G-03] | Consider using assembly to check for address(0)       |     3     |       18        |
| [G-04] | Consider using assembly to write storage values       |    10     |       660       |

Total: 73 instances over 4 issues with **~2529 gas** saved.
Per instance gas savings are listed in the individual issue description.

### [G-01] Cheaper Comparison

When comparing integers, it is cheaper to use strict `>` & `<` operators over `>=` & `<=` operators, even if you must increment or decrement one of the operands. Note: before using this technique, it's important to consider whether incrementing/decrementing one of the operators could result in an over/underflow.
This optimization is applicable when the optimizer is turned off.

Example:

```js
function gte() external pure returns (bool) {
    return 2 >= 2;
}

function gtPlusMinusOne() external pure returns (bool) {
    return 2 > 2 - 1;
}

function lte() external pure returns (bool) {
    return 2 <= 2;
}

function ltPlusOne() external pure returns (bool) {
    return 2 < 2 + 1;
}
```

Saved per instance: 3 gas

Lines:

- [StablecoinBridge.sol:50](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L50)
- [StablecoinBridge.sol:51](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L51)
- [Position.sol:53](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L53)
- [Position.sol:110](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L110)
- [Position.sol:184](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L184)
- [Position.sol:305](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L305)
- [Position.sol:307](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L307)
- [Position.sol:374](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L374)
- [Frankencoin.sol:141](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L141)
- [Frankencoin.sol:282](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L282)
- [Frankencoin.sol:294](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L294)
- [MintingHub.sol:109](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L109)
- [MintingHub.sol:171](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L171)
- [MintingHub.sol:172](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L172)
- [MintingHub.sol:201](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L201)
- [MintingHub.sol:218](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L218)
- [MintingHub.sol:255](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L255)

### [G-02] Consider using assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

Example:

```js
//addition in Solidity
function addTest(uint256 a, uint256 b) public pure {
    uint256 c = a + b;
}

//addition in assembly
function addAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := add(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}

//subtraction in Solidity
function subTest(uint256 a, uint256 b) public pure {
    uint256 c = a - b;
}

//subtraction in assembly
function subAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := sub(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}

//multiplication in Solidity
function mulTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}

//multiplication in assembly
function mulAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := mul(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}

//division in Solidity
function divTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}

//division in assembly
function divAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := div(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
```

Saved per instance:

- add: 40 gas
- sub: 37 gas
- mul: 60 gas
- div: 60 gas

Lines:

- [Position.sol:64](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L64)
- [Position.sol:66](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L66)
- [Position.sol:80](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L80)
- [Position.sol:98](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L98)
- [Position.sol:99](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L99)
- [Position.sol:100](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L100)
- [Position.sol:122](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L122)
- [Position.sol:124](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L124)
- [Position.sol:138](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L138)
- [Position.sol:142](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L142)
- [Position.sol:146](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L146)
- [Position.sol:150](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L150)
- [Position.sol:187](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L187)
- [Position.sol:194](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L194)
- [Position.sol:203](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L203)
- [Position.sol:241](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L241)
- [Position.sol:283](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L283)
- [Position.sol:307](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L307)
- [Frankencoin.sol:25](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L25)
- [Frankencoin.sol:88](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L88)
- [Frankencoin.sol:118](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L118)
- [Frankencoin.sol:144](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L144)
- [Frankencoin.sol:166](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L166)
- [Frankencoin.sol:168](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L168)
- [Frankencoin.sol:169](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L169)
- [Frankencoin.sol:196](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L196)
- [Frankencoin.sol:205](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L205)
- [Frankencoin.sol:209](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L209)
- [Frankencoin.sol:227](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L227)
- [Frankencoin.sol:238](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L238)
- [Frankencoin.sol:239](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L239)
- [Frankencoin.sol:253](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L253)
- [Frankencoin.sol:254](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L254)
- [Frankencoin.sol:286](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L286)
- [ERC20.sol:132](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L132)
- [MintingHub.sol:165](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L165)
- [MintingHub.sol:189](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L189)
- [MintingHub.sol:217](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L217)
- [MintingHub.sol:263](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L263)
- [MintingHub.sol:265](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L265)
- [MintingHub.sol:266](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L266)
- [MintingHub.sol:268](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L268)
- [MintingHub.sol:270](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L270)

### [G-04] Consider using assembly to check for address(0)

Example:

```js
contract Contract0 {
    function ownerNotZero(address _addr) public pure {
        require(_addr != address(0), "zero address)");
    }
}

function assemblyOwnerNotZero(address _addr) public pure {
    assembly {
        if iszero(_addr) {
            mstore(0x00, "zero address")
            revert(0x00, 0x20)
        }
    }
}
```

Saved per instance: 6 gas

Lines:

- [ERC20PermitLight.sol:56](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L56)
- [ERC20.sol:152](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L152)
- [ERC20.sol:180](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L180)

### [G-05] Consider using assembly to write storage values

Example:

```js
contract Contract0 {
    address owner = 0xb4c79daB8f259C7Aee6E5b2Aa729821864227e84;

    function updateOwner(address newOwner) public {
        owner = newOwner;
    }
}

contract Contract1 {
    address owner = 0xb4c79daB8f259C7Aee6E5b2Aa729821864227e84;

    function assemblyUpdateOwner(address newOwner) public {
        assembly {
            sstore(owner.slot, newOwner)
        }
    }
}
```

Saved per instance: 66 gas

Lines:

- [Position.sol:57](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L57)
- [Position.sol:65](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L65)
- [Position.sol:67](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L67)
- [Position.sol:80](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L80)
- [Position.sol:82](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L82)
- [Position.sol:112](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L112)
- [Position.sol:143](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L143)
- [Position.sol:165](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L165)
- [Position.sol:205](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L205)
- [Position.sol:272](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L272)
