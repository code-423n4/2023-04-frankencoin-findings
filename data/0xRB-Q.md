## constants should be defined rather than using magic numbers

Using magic numbers, which are numeric literals used in the code without any explanation of their meaning, can make programs harder to read and maintain. To improve code readability and maintainability, **constants** should be defined instead of magic numbers.

There are various instances of this issue:

> *File: contracts/Frankencoin.sol*
> 118: return minterReserveE6 / 1000000;
> 239: return 1000000 * amountExcludingReserve / (1000000 - adjustedReservePPM);

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L118
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L239

> *File: contracts/Position.sol*
> 122: return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
> 124: return totalMint * (1000_000 - reserveContribution) / 1000_000;

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L122

