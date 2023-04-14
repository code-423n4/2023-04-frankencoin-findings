## 1) Missing checks for address(0x0) when assigning values to address state variables

The contract **Equity.sol** should have a base case to the **canVoteFor** function to return false if delegate is the null address (address(0x0)), since in that case there is no delegate to vote for and the function can immediately return false without further recursion. This can save gas in cases where the delegate address is not set.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L225


## 2) It costs more gas to initialize variables with their default value than letting the default value be applied.

Explicitly initializing variables with its default values wastes around 3 gas per instance.

> *File: contracts/Equity.sol*
> 192: for (uint i=0; i<helpers.length; i++)
> 312: for (uint256 i = 0; i<addressesToWipe.length; i++)

## 3) Eliminate unnecessary operations in calculateProceeds function

In the Equity contract, An optimization can be made to this function by directly using the value of totalSupply(), instead of computing totalShares and newTotalShares separately. This can save gas by avoiding an extra computation step.

The change would be:

>

     function calculateProceeds(uint256 shares) public view returns (uint256) {
        uint256 capital = zchf.equity();
        require(shares + ONE_DEC18 < totalSupply(), "too many shares"); // make sure there is always at least one share
        uint256 newCapital = _mulD18(capital, _power3(_divD18(totalSupply() - shares, totalSupply())));
        return capital - newCapital;
    }

