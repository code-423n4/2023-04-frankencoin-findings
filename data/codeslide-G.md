### Gas Optimizations

| Number | Issue                            | Instances |
| :----: | :------------------------------- | :-------: |
| [G-01] | State variables should be cached |     5     |
| [G-02] | Unnecessary computation          |     2     |

#### [G-01] State variables should be cached

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read.

Saves 100 gas per instance

```solidity
File: contracts/Frankencoin.sol

// 2nd call to totalSupply() which reads _totalSupply
85:    if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
```

```solidity
File: contracts/Position.sol

// mintingFeePPM and start
187:    return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));

// minted
241:    if (amount > minted) revert RepaidTooMuch(amount - minted);

// minted
349:    uint256 repayment = minted < volumeZCHF ? minted : volumeZCHF;
```

#### [G-02] Unnecessary computation

1. The call to `zchf.equity()` on line 292 is not necessary if the `require()` statement on 293 evaluates to `false`. Recommended to swap lines 292 and 293.

   ```solidity
   File: contracts/Equity.sol

   291:    uint256 totalShares = totalSupply();
   292:    uint256 capital = zchf.equity();
   293:    require(shares + ONE_DEC18 < totalShares, "too many shares");
   ```

1. The `require()` at line 109 should be moved to the first line of the `openPosition()` function to avoid any unnecessary code execution.

   ```solidity
   File: contracts/MintingHub.sol

   109:    require(_initialCollateral >= _minCollateral, "must start with min col");
   ```
