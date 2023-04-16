# **Position.sol**

## 1.Rewrite calculation in *reduceLimitForClone*

A total of 198 gas can be saved by rewriting the function as follows:

    function reduceLimitForClone(uint256 _minimum) external noChallenge noCooldown alive onlyHub returns (uint256)      {
        uint256 result = (limit - minted - _minimum)/2 + _minimum;
        limit -= result;
        return result; 
    }

## 2.Use private for constants 

By making constants private instead of public a total of 30676 gas is saved during deployment of the contract: 

    uint256 private constant MIN_FEE = 1000 * (10**18);