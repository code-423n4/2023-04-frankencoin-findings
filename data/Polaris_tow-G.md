## CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS
Make it outside and only use it inside.
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L192-L202
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L312-L316

```
        for (uint256 i = 0; i<addressesToWipe.length; i++){
            address current = addressesToWipe[0];
            _burn(current, balanceOf(current));
        }
    }
```
## REQUIRE() OR REVERT() STATEMENTS THAT CHECK INPUT ARGUMENTS SHOULD BE AT THE TOP OF THE FUNCTION
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. 
By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L253
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L293
```
    function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool) {
        require(msg.sender == address(zchf), "caller must be zchf");
        uint256 equity = zchf.equity();
        require(equity >= MINIMUM_EQUITY, "insuf equity"); // ensures that the initial deposit is at least 1000 ZCHF


        // Assign 1000 FPS for the initial deposit, calculate the amount otherwise
        uint256 shares = equity <= amount ? 1000 * ONE_DEC18 : calculateSharesInternal(equity - amount, amount);
        _mint(from, shares);
        emit Trade(msg.sender, int(shares), amount, price());


        // limit the total supply to a reasonable amount to guard against overflows with price and vote calculations
        // the 128 bits are 68 bits for magnitude and 60 bits for precision, as calculated in an above comment
        require(totalSupply() < 2**128, "total supply exceeded");
        return true;
    }
```














