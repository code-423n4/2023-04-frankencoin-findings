# Possible Data Truncation

#### There are two instances of this issue in MathUtil.sol
~~~
   function _mulD18(uint256 _a, uint256 _b) internal pure returns(uint256) {
       return _a * _b / ONE_DEC18;
   }
~~~

 https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L32

~~~
   function _divD18(uint256 _a, uint256 _b) internal pure returns(uint256) {
        return (_a * ONE_DEC18) / _b ;
    }
~~~
 https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L36


The data might get truncated as both _a and _b are 256-bit unsigned integers and the multiplication operation  can may exceed the maximum value that can be represented by a 256-bit unsigned integer. When the result of the multiplication exceeds the maximum value of a 256-bit unsigned integer, the result will wrap around and may not be what is expected. This is commonly referred to as an overflow

Similarly, the division operation in the code snippet may also cause truncation of data if the result of the  multiplication operation is not a multiple of ONE_DEC18. In this case, the result will be rounded down to the  nearest integer value.
