## Low Risk Findings
### [L-01] Modify calculations of x^3 to perform 1e36 division after multiplication to minimize rounding errors

Currently, the `_cubicRoot` function in `MathUtil` performs the following calculation:

```solidity
uint256 powX3 = _mulD18(_mulD18(x, x), x);
```
however some precision is lost due to division by the 1e18 scaling factor at the intermediate x^2 step. This can be fixed by performing the division of 1e36 after the multiplication, using a 512-bit fixed-point math library to avoid overflow for the case of large x above 1e25.

### [L-02] Refund excess in `Position::notifyRepaidInternal` rather than reverting

Currently, the `Position::notifyRepaidInternal` function in `Position` performs the following validation:

```solidity
if (amount > minted) revert RepaidTooMuch(amount - minted);
```
however a better approach would be to refund the excess amount to the caller, as this would allow the caller to guarantee that their transaction will repay the debt.

### [L-03] Wrong value of seconds in year

The `StablecoinBridge` contract in `StablecoinBridge` has a constructor that performs the following calculation:

```solidity
```
which is used to specify the time horizon after which the bridge expires and needs to be replaced by a new contract.

Most precise calculations take into account leap years (typically 365.25 days); however, as can be seen at [NASA](https://pumas.nasa.gov/sites/default/files/examples/04_21_97_1.pdf) and [stackoverflow](https://astronomy.stackexchange.com/questions/21182/to-several-decimal-places-how-many-days-are-in-one-year), the value is slightly different.

Current case:
52 weeks = 364 days = 31_449_600 / (24 * 3600)
Naive case:
365.25 days = 31_557_600 / (24 * 3600)
NASA case:
365.2422 days = 31_556_926 / (24 * 3600)

Here, use of 52 weeks in Solidity will underestimate the time after which the contract should be replaced by approximately a day. Use 31_556_926 seconds in one year for maximum precision: `horizon = block.timestamp + 31_556_926 seconds;`

## Non-Critical Findings
### [NC-01] Use `MathUtil::_power3` in `MathUtil::_cubicRoot`

The `MathUtil` contract defines a function to raise an input value to its third power; however, it is not used in the `MathUtil::_cubicRoot` function. This can be fixed by replacing the following line:

```solidity
uint256 powX3 = _mulD18(_mulD18(x, x), x);
```
with
```solidity
uint256 powX3 = _power3(x);
```

### [NC-02] Use a constant for `1000000`

The `Frankencoin` contract uses the value `1000000` inlined in calculations. It would be better to use a constant instead:

```solidity
uint256 internal constant ONE_MILLION = 1_000_000;
```

### [NC-03] Use `1e18` & `1e16` as constants

The `MathUtil` contract has the following constants:

```solidity
uint256 internal constant ONE_DEC18 = 10**18;
uint256 internal constant THRESH_DEC18 =  10000000000000000;
```

It would be more readable and consistent to use the following constants instead:

```solidity
uint256 internal constant ONE_DEC18 = 1e18;
uint256 internal constant THRESH_DEC18 = 1e16;
```

### [NC-04] Remove unnecessary parentheses in `MathUtil::_divD18`

Due to the oder of operator precedence, the parentheses in the following line are unnecessary:

```solidity
return (_a * ONE_DEC18) / _b ;
```
