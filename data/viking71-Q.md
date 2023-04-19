# [L1] Two step ownership transfer
`Ownable`'s `transferOwnership` is not recommended. Instead use the `Ownable2Step`'s `transferOwnership` for 2 step change. Through this only when the new owner accepts the ownership the process is finished. 

## Code Snippet
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol

## Reference
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol