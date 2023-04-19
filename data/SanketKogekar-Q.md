
### Equity.sol -

1. In Equity.sol, the IERC677Receiver interface is imported, but never used.
    https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L7

### MintingHub.sol -

2. In MintingHub.sol, the `openPosition()` function transfers tokens from the caller to the newly created position, without checking whether the caller has enough tokens to cover the transfer. This could result in the loss of tokens if the caller doesn't have enough.
    https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L108

3. In MintingHub.sol, the function `openPosition()` does not validate the input parameters, such as checking if the `_minCollateral` is greater than zero, if `_liqPrice` is non-zero, or `_mintingFeePPM` is a large number.

4. In MintingHub.sol, the `challenges` array can grow indefinitely, which could cause scalability issues if many challenges are open at once.

5. In MintingHub.sol, the `openPosition()` does not check whether the `_mintingMaximum `parameter is greater than zero, which could allow someone to create a position that mints 0 ZCHF.

6. There is no check to prevent someone from creating a challenge for a position that has already expired.

### PositionFactory.sol -

7. The `createNewPosition()` function allows creating invalid positions. It should verify that `_duration` is greater than `_challengePeriod`, also `_duration` and `_mintingFeePPM` are within a certain range.

8. In the `createClone()` function, the `targetBytes` variable is used to hold the first 20 bytes of the target address, but it is not ensured that the remaining bytes of the target address are zero. Just to be safe.

9. The `clonePosition()` function calls the `createClone()` function with the `existing.original()` address as the argument, but it is not checked for a valid address.

10. The `createClone()` function does not check the return value of the `create()` function. If the clone creation fails, the function will continue to execute as if the clone was created successfully.

11. The `createNewPosition()` does not perform any input validation on the arguments passed in. The _mintingFeePPM parameter could be set to a value greater than the max allowed, which could result in incorrect fee calculations.
