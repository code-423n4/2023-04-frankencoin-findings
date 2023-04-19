# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1 | Collateral invarient not checked in some cases | Low | 2 |
| 2 | Missing zero address check in `setOwner` function | Low | 1 |
| 3 | No zero address check on `sender` in `_transfer` function | Low | 1 |
| 4 | Check that `mintingFeePPM` and `reserveContribution` are less than 1e6 | Low |  |
| 5 | Immutable state variables lack zero address checks | Low | 6 |
| 6 | Missing Natspec/comments in many functions | NC | 5 |
| 7 | Use scientific notation | NC | 8 |

## Findings

### 1- Collateral invarient not checked in some cases :

#### Risk : Low

In the Position contract there is a collateral invarient that must always hold and should be checked whenever one of these values changes : price, collateral balance and minted; This is explained in `checkCollateral` function comments :

File: Position.sol [Line 278-281](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L278-L281)
```solidity
/**
* This invariant must always hold and must always be checked when any of the three
* variables change in an adverse way.
*/
```

There are two instances where this invarient is not checked : 

1- when the `adjustPrice` function is called if the new price `newPrice` is greater than the old price `price` the invarient is not checked, instead a cooldown period is triggered and the new price is set immediatly.

File: Position.sol [Line 159-166](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L159-L166)
```solidity
function adjustPrice(uint256 newPrice) public onlyOwner noChallenge {
    /** @audit
        checkCollateral only called if newPrice <= price
    */
    if (newPrice > price) {
        restrictMinting(3 days);
    } else {
        checkCollateral(collateralBalance(), newPrice);
    }
    price = newPrice;
    emitUpdate();
}
```

2- The `notifyChallengeSucceeded` function does call the functions `notifyRepaidInternal` and `internalWithdrawCollateral` which decreases the values of `minted` and collateral balance respectively, so the colllateral invarient should be checked but it is not.

File: Position.sol [Line 329-354](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L329-L354)
```solidity
function notifyChallengeSucceeded(
    address _bidder,
    uint256 _bid,
    uint256 _size
) external onlyHub returns (address, uint256, uint256, uint256, uint32) {
    challengedAmount -= _size;
    uint256 colBal = collateralBalance();
    if (_size > colBal) {
        // Challenge is larger than the position. This can for example happen if there are multiple concurrent
        // challenges that exceed the collateral balance in size. In this case, we need to redimension the bid and
        // tell the caller that a part of the bid needs to be returned to the bidder.
        _bid = _divD18(_mulD18(_bid, colBal), _size);
        _size = colBal;
    }

    // Note that thanks to the collateral invariant, we know that
    //    colBal * price >= minted * ONE_DEC18
    // and that therefore
    //    price >= minted / colbal * E18
    // such that
    //    volumeZCHF = price * size / E18 >= minted * size / colbal
    // So the owner cannot maliciously decrease the price to make volume fall below the proportionate repayment.
    uint256 volumeZCHF = _mulD18(price, _size); // How much could have minted with the challenged amount of the collateral
    // The owner does not have to repay (and burn) more than the owner actually minted.
    uint256 repayment = minted < volumeZCHF ? minted : volumeZCHF; // how much must be burned to make things even

    /** @audit
        `minted` and collateral balance are changed after this calls
    */
    notifyRepaidInternal(repayment); // we assume the caller takes care of the actual repayment
    internalWithdrawCollateral(_bidder, _size); // transfer collateral to the bidder and emit update
    return (owner, _bid, volumeZCHF, repayment, reserveContribution);
}
```


#### Mitigation
Make sure that the `checkCollateral` check is not necessary in the instances aferomentioned, if not the `checkCollateral` check must be added.


### 2- Missing zero address check in `setOwner` function  :

#### Risk : Low

The function `setOwner` is used to transfer the ownership in the `Ownable` contract, but the function is missing a zero address check on the new owner address `newOwner` which could lead to a loss of ownership (it can be burned by accident during the transfer), this issue should be considered and addressed as many contracts of the protocol rely on the `Ownable` contract for managing ownership.

#### Mitigation
Consider adding a non zero address check in the `setOwner` function, or to make it even safer you should user 2-step onwership transfer pattern.

### 3- No zero address check on `sender` in `_transfer` function :

#### Risk : Low

In the `_transfer` function inside the `ERC20` contract there is no non-zero address check for the `sender` address even though the rquirements in the function comments indicates that it should not be `address(0)` :

File: ERC20.sol [Line 145-147](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L145-L147)
```
* Requirements:
*
* - `sender` cannot be the zero address.
```

This can potentialy cause problems especially in the `transferFrom` function as it also does not check the sender` address 

#### Mitigation
Add a non zero address check for the `sender` address in the `_transfer` function.

### 4- Check that `mintingFeePPM` and `reserveContribution` are less than 1e6 :

#### Risk : Low

The variables `mintingFeePPM` and `reserveContribution` represent the minting fee and the reserve percentage, both variables are in PPM meaning that 1e6 ≈ 100%, because they are both immutable and thus cannot be changed after deployment the contract should check if their values are indeed less than 1e6 to avoid any problems later.

#### Mitigation
Add checks in the constructor of the Position contract to verify that the values of `mintingFeePPM` and `reserveContribution` are less than 1e6


### 5- Immutable state variables lack zero address checks :

Constructors should check the values written in an immutable state variables(address) is not the address(0).

#### Risk : Low

#### Proof of Concept
Instances include:

File: StablecoinBridge.sol [Line 27-28](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L27-L28)
```
chf = IERC20(other);
zchf = IFrankencoin(zchfAddress);
```

File: MintingHub.sol [Line 55-56](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L55-L56)
```
zchf = IFrankencoin(_zchf);
POSITION_FACTORY = IPositionFactory(factory);
```

File: Position.sol [Line 58-59](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L58-L59)
```
zchf = IFrankencoin(_zchf);
collateral = IERC20(_collateral);
```

#### Mitigation
Add non-zero address checks in the constructors for the instances aforementioned.

### 6- Missing Natspec/comments in many functions :

#### Risk : Non critical

Consider adding natspec/comments on all functions to improve code documentation and readability.

Instances include :

File: Equity.sol 

[Lines 190-202](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L190-L202)

[Lines 225-233](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L225-L233)

[Lines 266-270](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L266-L270)


File: Position.sol 

[Lines 181-189](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L181-L189)

[Lines 292-296](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L292-L296)


### 7- Use scientific notation :

When using multiples of 10 you shouldn't use decimal literals or exponentiation (e.g. 1000000, 10**18) but use scientific notation for better readability.

#### Risk : Non critical

#### Proof of Concept

Instances include:

File: MathUtil.sol [Line 11](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L11)
```
uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01
```

File: Frankencoin.sol 

[Line 118](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L118)
```
return minterReserveE6 / 1000000;
```

[Line 166](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L166)
```
uint256 usableMint = (_amount * (1000_000 - _feesPPM - _reservePPM)) / 1000_000;
```

[Line 205](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L205)
```
uint256 theoreticalReserve = _reservePPM * mintedAmount / 1000000;
```

[Line 239](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L239)
```
return 1000000 * amountExcludingReserve / (1000000 - adjustedReservePPM);
```

File: MintingHub.sol 

[Line 265](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L265)
```
uint256 reward = (volume * CHALLENGER_REWARD) / 1000_000;
```

File: Position.sol 

[Line 122](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L122)
```
return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
```

[Line 124](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L124)
```
return totalMint * (1000_000 - reserveContribution) / 1000_000;
```

#### Mitigation
Use scientific notation for the instances aforementioned.