## Remove unused imports
PositionFactory.sol - [5](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L5)

StablecoinBridge.sol - [5](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L5)

Equity.sol - [7](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L7)

MintingHub.sol - [5](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L5), [7](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L7)

## No need of `else`/`else-if` block
Frankencoin.sol - [106](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L106), [108](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L108), [143](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L143), [210](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L210)

Equity.sol - [163](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L163), [228](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L228), [230](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L230)

Position.sol - [123](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L123), [186](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L186), [307](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L307), [314](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L314)

## Missing zero address checks
ERC20.sol - [200](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L200)

MintingHub.sol - [54](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L54)

Position.sol - [50](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L50)

## Missing revert messages
Equity.sol - [194](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L194), [195](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L195), [197](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L197), [276](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L276),  [310](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L310)

MintingHub.sol - [158](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L158), [171](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L171), [172](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L172), [254](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L254)

Position.sol - [53](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L53)

## `ERC20` not following EIP20 standard
[ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol)

## Constants Should Be Defined Rather Than Using Magic Numbers
MintingHub.sol - [265](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L265)

Position.sol - [122](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L122), [124](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L124)

Frankencoin.sol - [118](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L118), [166](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L166), [205](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L205), [239](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L239)

Equity.sol - [211](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L211), [247](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L247), [268](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L268)

## Error defined but never used
Position.sol - [104](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L104)

## Missing checks if `amount` is 0
ERC20.sol - [179](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L179), [200](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L200),