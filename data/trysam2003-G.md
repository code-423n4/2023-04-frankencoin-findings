## 1. && costs more gas than a separate expression 
The && operator is a logical AND operator that checks if both of its operands evaluate to true. Using the && operator can be more expensive regarding gas costs than using separate expressions because it requires additional computational overhead to evaluate the operands and check the result. 
#### Proof of Concept

        84: if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
        85: if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

## 2. X>0 consumes more gas than X!=0
This is because X>0 requires the EVM to perform an additional operation with 0, which can consume slightly more gas than the X!=0 operator, which simply checks whether the value is non-zero.

#### Proof of Concept
        104: if (explicit > 0)      
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

        203 if (challenge.bid > 0) {
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol 

## 3. x<=y consumes more gas than  x>y. Also, x>=y consumes more gas than  x<y 
This is because the <= operator requires the EVM to perform an additional comparison operation to check if the value is equal, while the > operator only needs to check if the value is greater. This additional comparison operation can consume more gas and result in slightly higher gas costs.

#### Proof of Concept
        141: if (balance <= minReserve) 
        282:if (reserveLeft >= _amount)     
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

        171: require(challenge.size >= min);
        172: require(copy.size >= min);
        201: if (block.timestamp >= challenge.end) revert TooLate();
        218: if (earliestEnd >= challenge.end) {      
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

       184: if (time >= exp)
       305: if (block.timestamp >= expiration){
       374: if (block.timestamp <= cooldown) revert Hot();
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol
      
       50:  require(block.timestamp <= horizon, "expired");
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol

       








