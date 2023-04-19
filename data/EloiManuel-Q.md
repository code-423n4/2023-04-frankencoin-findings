## Summary

Note that this is my first submission at Code4rena and I am still figuring out how things work. I could not spend much time on this.

### Issues
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low Risk | Potential risk |
| NC |  Non-critical | Non risky findings |

| Total Found Issues | 5 |
|:--:|:--:|

### Low Risk Issues
| Count | Explanation |
|:--:|:-------:|
| [L-01] | `restructureCapTable()` incorrect burning mechanism |

| Total Low Risk Issues | 1 |
|:--:|:--:|

### Non-critical Risk Issues
| Count | Explanation |
|:--:|:-------:|
| [NC-01] | Missing or vague error messages |
| [NC-02] | Constants should be defined rather than using magic numbers |
| [NC-03] | Inconsistent Documentation |
| [NC-04] | Unnecessary Import |

| Total Non-critical Risk Issues | 4 |
|:--:|:--:|

### [L-01] `restructureCapTable()` incorrect burning mechanism
The function `restructureCapTable()` in `Equity.sol` is supposed to burn the balance of all addresses in `addressesToWipe[]`. However, it would only do it for the first element as per line 313 shown below:

```solidity
contracts/Equity.sol

309:    function restructureCapTable(address[] calldata helpers, address[] calldata addressesToWipe) public {
310:    require(zchf.equity() < MINIMUM_EQUITY);
311:    checkQualified(msg.sender, helpers);
312:    for (uint256 i = 0; i<addressesToWipe.length; i++){
313:        address current = addressesToWipe[0];
314:        _burn(current, balanceOf(current));
315:    }
```

Note that in the event using `addressesToWipe` with a considerable number of elements, it would only burn the tokens of the first address though using a lot of gas.

#### Recommended Mitigation Steps
Change `address current = addressesToWipe[0];` to `address current = addressesToWipe[i]`.


### [NC-01] Missing or vague error messages
Some `require` statements do not have an error message or are the error messages they have are not clear.

```solidity
contracts/Position.sol

53: require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values
```

```solidity
contracts/Equity.sol

276: require(canRedeem(msg.sender));
``` 

```solidity
contracts/StablecoinBridge.sol

51:    require(block.timestamp <= horizon, "expired");
52:    require(chf.balanceOf(address(this)) <= limit, "limit");
```

### [NC-02] Constants should be defined rather than using magic numbers
A magic number is a numeric literal that is used in the code without any explanation of its meaning. The use of magic numbers makes programs less readable and hence more difficult to maintain and update. Even assembly can benefit from using readable constants instead of hex/numeric literals

```solidity
contracts/Frankencoin.sol

118:      return minterReserveE6 / 1000000;

166:      uint256 usableMint = (_amount * (1000_000 - _feesPPM - _reservePPM)) / 1000_000; // rounding down is fine

205:      uint256 theoreticalReserve = _reservePPM * mintedAmount / 1000000;

239:      return 1000000 * amountExcludingReserve / (1000000 - adjustedReservePPM); // 41 / (1-18%) = 50
```

### [NC-03] Inconsistent Documentation

The documentation (https://docs.frankencoin.com/positions/open) says that this is the function to open a position:
```solidity
	openPosition(
			address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
			uint256 _mintingMaximum, uint256 _expirationSeconds, uint256 _challengeSeconds,
			uint32 _mintingFeePPM, uint256 _liqPriceE18, uint32 _reservePPM)
```

It also says: "When opening a position, it is already provided with some collateral, but nothing is minted yet.
Minting is not possible until the initialization period of seven days has passed." However, MintingHub also has:

```solidity
    function openPosition(
        address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
        uint256 _mintingMaximum, uint256 _initPeriodSeconds, uint256 _expirationSeconds, uint256 _challengeSeconds,
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) 
```
which let's the user choose another value other than 7 days (up to 3 days or more).

Furthermore, on Equity.sol, it says that equity could be negative when it is using a `uint`:

```solidity
300:     * If there is less than 1000 ZCHF in equity left (maybe even negative),
```

### [NC-04] | Unnecessary Import
The following import in `PositionFactory.sol` was not found to be necessary:

```solidity
5:  import "./IFrankencoin.sol";
```