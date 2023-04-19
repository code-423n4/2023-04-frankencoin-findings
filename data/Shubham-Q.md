## Low Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| [L-01](#L-01) | Update codes to avoid Compile Errors | 43 |
| [L-02](#L-02) | In the constructor, there is no return of incorrect address identification | 3 |

*Total 2 issues.*

## Non-Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Lines too long (apart from automated ones) | 3 |
| [NC-2](#NC-2) | Assembly Codes Specific – Should Have Comments | 1 |
| [NC-3](#NC-3) | Contracts do not comply with `the Solidity Style Guide` | 1 |

*Total 3 issues.*

## [L-01] Update codes to avoid Compile Errors

*There are 43 instances of this issue.*

```solidity
Warning: This declaration shadows an existing declaration.
   --> contracts/test/MintingHubTest.sol:326:14:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |              ^^^^^^^^^^^^^^^^^^
Note: The shadowed declaration is here:
   --> contracts/test/MintingHubTest.sol:331:10:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |          ^^^^^^^^^^^^^^^^^^


Warning: This declaration shadows an existing declaration.
   --> contracts/test/MintingHubTest.sol:326:34:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                  ^^^^^^^^^^^
Note: The shadowed declaration is here:
   --> contracts/test/MintingHubTest.sol:331:30:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                              ^^^^^^^^^^^


Warning: This declaration shadows an existing declaration.
   --> contracts/test/MintingHubTest.sol:326:47:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                               ^^^^^^^^^^^^
Note: The shadowed declaration is here:
   --> contracts/test/MintingHubTest.sol:331:43:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                           ^^^^^^^^^^^^


Warning: This declaration shadows an existing declaration.
   --> contracts/test/MintingHubTest.sol:326:61:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                             ^^^^^^^^^
Note: The shadowed declaration is here:
   --> contracts/test/MintingHubTest.sol:331:57:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                         ^^^^^^^^^


Warning: This declaration shadows an existing declaration.
   --> contracts/test/MintingHubTest.sol:326:72:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                        ^^^^^^^^^
Note: The shadowed declaration is here:
   --> contracts/test/MintingHubTest.sol:331:68:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                    ^^^^^^^^^


Warning: This declaration shadows an existing declaration.
   --> contracts/test/MintingHubTest.sol:326:83:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                                   ^^^^^^^^^^^
Note: The shadowed declaration is here:
   --> contracts/test/MintingHubTest.sol:331:79:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                               ^^^^^^^^^^^


Warning: This declaration shadows an existing declaration.
   --> contracts/test/MintingHubTest.sol:331:79:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                               ^^^^^^^^^^^
Note: The shadowed declaration is here:
   --> contracts/test/MintingHubTest.sol:336:5:
    |
336 |     function bid(MintingHub hub, uint256 number, uint256 amount) public {
    |     ^ (Relevant source part starts here and spans across multiple lines).


Warning: This declaration shadows an existing declaration.
   --> contracts/test/MintingHubTest.sol:337:79:
    |
337 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(number);
    |                                                                               ^^^^^^^^^^^
Note: The shadowed declaration is here:
   --> contracts/test/MintingHubTest.sol:336:5:
    |
336 |     function bid(MintingHub hub, uint256 number, uint256 amount) public {
    |     ^ (Relevant source part starts here and spans across multiple lines).


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:100:30:
    |
100 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(latestChallenge);
    |                              ^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:100:43:
    |
100 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(latestChallenge);
    |                                           ^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:100:57:
    |
100 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(latestChallenge);
    |                                                         ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:100:68:
    |
100 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(latestChallenge);
    |                                                                    ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:118:11:
    |
118 |          (address challenger1, IPosition p1, uint256 size1, uint256 a1, address b1, uint256 bid1) = hub.challenges(number);
    |           ^^^^^^^^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:118:32:
    |
118 |          (address challenger1, IPosition p1, uint256 size1, uint256 a1, address b1, uint256 bid1) = hub.challenges(number);
    |                                ^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:118:61:
    |
118 |          (address challenger1, IPosition p1, uint256 size1, uint256 a1, address b1, uint256 bid1) = hub.challenges(number);
    |                                                             ^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:118:73:
    |
118 |          (address challenger1, IPosition p1, uint256 size1, uint256 a1, address b1, uint256 bid1) = hub.challenges(number);
    |                                                                         ^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:141:57:
    |
141 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(number);
    |                                                         ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:141:68:
    |
141 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(number);
    |                                                                    ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:217:10:
    |
217 |         (address challenger, IPosition p, uint256 size, uint256 end, address b, uint256 bid) = hub.challenges(latestChallenge);
    |          ^^^^^^^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:217:30:
    |
217 |         (address challenger, IPosition p, uint256 size, uint256 end, address b, uint256 bid) = hub.challenges(latestChallenge);
    |                              ^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:217:43:
    |
217 |         (address challenger, IPosition p, uint256 size, uint256 end, address b, uint256 bid) = hub.challenges(latestChallenge);
    |                                           ^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:217:70:
    |
217 |         (address challenger, IPosition p, uint256 size, uint256 end, address b, uint256 bid) = hub.challenges(latestChallenge);
    |                                                                      ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:217:81:
    |
217 |         (address challenger, IPosition p, uint256 size, uint256 end, address b, uint256 bid) = hub.challenges(latestChallenge);
    |                                                                                 ^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:221:10:
    |
221 |         (address challenger2, IPosition p2, uint256 size2, uint256 end2, address b2, uint256 bid2) = hub.challenges(latestChallenge);
    |          ^^^^^^^^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:221:31:
    |
221 |         (address challenger2, IPosition p2, uint256 size2, uint256 end2, address b2, uint256 bid2) = hub.challenges(latestChallenge);
    |                               ^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:221:45:
    |
221 |         (address challenger2, IPosition p2, uint256 size2, uint256 end2, address b2, uint256 bid2) = hub.challenges(latestChallenge);
    |                                             ^^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:221:74:
    |
221 |         (address challenger2, IPosition p2, uint256 size2, uint256 end2, address b2, uint256 bid2) = hub.challenges(latestChallenge);
    |                                                                          ^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:221:86:
    |
221 |         (address challenger2, IPosition p2, uint256 size2, uint256 end2, address b2, uint256 bid2) = hub.challenges(latestChallenge);
    |                                                                                      ^^^^^^^^^^^^


Warning: Unused function parameter. Remove or comment out the variable name to silence this warning.
   --> contracts/test/MintingHubTest.sol:272:19:
    |
272 |     function deny(MintingHub hub, address pos) public {
    |                   ^^^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:326:14:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |              ^^^^^^^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:326:61:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                             ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:326:72:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                        ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:326:83:
    |
326 |             (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                                   ^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:331:30:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                              ^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:331:43:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                           ^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:331:57:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                         ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:331:68:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                    ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:331:79:
    |
331 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(first);
    |                                                                               ^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:337:10:
    |
337 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(number);
    |          ^^^^^^^^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:337:30:
    |
337 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(number);
    |                              ^^^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:337:57:
    |
337 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(number);
    |                                                         ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:337:68:
    |
337 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(number);
    |                                                                    ^^^^^^^^^


Warning: Unused local variable.
   --> contracts/test/MintingHubTest.sol:337:79:
    |
337 |         (address challenger, IPosition p, uint256 size, uint256 a, address b, uint256 bid) = hub.challenges(number);
    |                                                                               ^^^^^^^^^^^

```

## [L-02] In the constructor, there is no return of incorrect address identification

In case of incorrect address definition in the constructor , there is no way to fix it because of the variables are immutable.

```solidity
File: main/contracts/MintingHub.sol

L:54        constructor(address _zchf, address factory) {
L:55           zchf = IFrankencoin(_zchf);
L:56           POSITION_FACTORY = IPositionFactory(factory);
L:57        }
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L54

```solidity
File: main/contracts/StablecoinBridge.sol

L:26       constructor(address other, address zchfAddress, uint256 limit_){
L:27          chf = IERC20(other);
L:28          zchf = IFrankencoin(zchfAddress);
L:29          horizon = block.timestamp + 52 weeks;
L:30          limit = limit_;
L:31       }
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L26

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L50-L67

## [NC-1] Lines too long

Lines in source code are typically limited to 80 characters, but it’s reasonable to stretch beyond this limit when need be as monitor screens theses days are comparatively larger. Considering the files will most likely reside in GitHub that will have a scroll bar automatically kick in when the length is over 164 characters, all code lines and comments should be split when/before hitting this length. Keep line width to max 120 characters for better readability where possible. Here are some of the instances entailed:
*There are 3 **more** instances of this issue.*

```solidity
File: main/contracts/Equity.sol

L:268     uint256 newTotalShares = totalShares < 1000 * ONE_DEC18 ? 1000 * ONE_DEC18 : _mulD18(totalShares, _cubicRoot(_divD18(capitalBefore + investment, capitalBefore)));
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L268

```solidity
File: main/contracts/MintingHub.sol

L:260      (address owner, uint256 effectiveBid, uint256 volume, uint256 repayment, uint32 reservePPM) = challenge.position.notifyChallengeSucceeded(recipient, challenge.bid, challenge.size);

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L260

```solidity
File: main/contracts/MintingHub.sol

L:329       function notifyChallengeSucceeded(address _bidder, uint256 _bid, uint256 _size) external onlyHub returns (address, uint256, uint256, uint256, uint32) {
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L329


## [NC-2] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

```solidity
File: main/contracts/PositionFactory.sol

L:39         assembly {
L:40            let clone := mload(0x40)
L:41            mstore(clone, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
L:42            mstore(add(clone, 0x14), targetBytes)
L:43            mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
L:44            result := create(0, clone, 0x37)
L:45         }
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L39-L45

## [NC-3] Contracts do not comply with `the Solidity Style Guide`

The below mentioned contracts do no follow **Order of Layout** section of the `Solidity Style Guide`.
https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout

**Contracts:** 

Equity.sol
Frankencoin.sol
MintingHub.sol
Ownable.sol
Position.sol
