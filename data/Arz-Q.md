## Title

Incorrect amount of shares can be minted because the cubicRoot function returns incorrect results if the given number is not a 1e+18

## Links

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol  
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol

## Brief/Introduction
The amount of shares can be calculated incorrectly if the given number is not a 1e+18 for the cubicRoot() function in Equity.sol. Users can get more than allowed shares.

## Explanation
If the totalShares < 1000 * ONE_DEC18, the newTotalShares are calculated using this equation:

_mulD18(totalShares, _cubicRoot(_divD18(capitalBefore + investment, capitalBefore))


The problem is when _divD18() returns a 1e19(or bigger) number for the cubicRoot function.

The cubicRoot function expects a 1e18 and returns incorrect results if the number given is not an 1e18 number.

This leads to incorrect calculation of the shares that the user will get and could leave the user with more shares than allowed.

## Impact
If _divD18() returns a number that is bigger(more zeros) than 1e18, the cubicRoot() function will receive an input that is not an 1e18 number, and will therefore return incorrect results



## Tests
In the test i did what Equity.sol is doing: _divD18(capitalBefore + investment, capitalBefore)) but i set investment to be a high number so when you _divD18() these 2 you get a 1e19 number and that number is then passed to the cubicRoot() which expects a 1e18 number but because the number is a 1e19 it returns incorrect results
```
 contract TestMathUtil is Test {

    MathUtil math;
    uint256 private constant MINIMUM_EQUITY = 1000 * ONE_DEC18;

    //Amount im transferring so i can make the divD18 return a 1e19 number
    uint256 private constant INVESTMENT = 9_000_000_000_000_000_000_000;

    function setUp() public {
        math = new MathUtil();


    }

    uint256 internal constant ONE_DEC18 = 1018;

    function testDivD18() public view  {
        //returns 1e19
        uint256 incorrectResult = math._divD18(MINIMUM_EQUITY + INVESTMENT, MINIMUM_EQUITY);

        //_cubicRoot(uint256 _v) expects _v to be a 1e18
        //this returns incorrect results
        console.log("THE BAD RESULT IS", math._cubicRoot(incorrectResult));


        //returns 1e18
        uint correctResult = math._divD18(MINIMUM_EQUITY, MINIMUM_EQUITY);

        //this returns the correct results
        console.log("THE CORRECT RESULT IS", math._cubicRoot(correctResult));
    }

}

```



## Recommendation
Review the function for proper adjustments
