## Gas Optimization Report:

### [G-01] ``<x> += <y>`` costs more than ``<x> = <x> + <y>`` for state variables.

## Code Block:

```
File: contracts/Position.sol
@audit limit on line 100

 function reduceLimitForClone(uint256 _minimum) external noChallenge noCooldown alive onlyHub returns (uint256) {
        uint256 reduction = (limit - minted - _minimum)/2; // this will fail with an underflow if minimum is too high
100:        limit -= reduction + _minimum;
        return reduction + _minimum;
    }
```

### Code Reference: 
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L100)


