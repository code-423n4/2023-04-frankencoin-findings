1) MintingHub contract's function openPosition()
   The below condition is checked in the overloaded function that created the position.
   Since the condition is based on input parameter, this check could be done much early in the flow of 
   the program.
   
   Idealy, create a modifier with the below require condition and attach to both openPosition functions.

   require(_initialCollateral >= _minCollateral, "must start with min col");



2) Deleting elements from arrays does not remove the entry from the array unless expensive copy over and 
   pop functions are implemented. instead of doing this, it would be better to mark the state of the 
   challenge as active, expired and concluded using an enum.

   Example:
   enum ChallengeStatus {
        ACTIVE,
        DEFEATED,
        SUCCESSFUL,
        DISQUALIFIED
   }
  
   struct Challenge {
        address challenger; // the address from which the challenge was initiated
        IPosition position; // the position that was challenged
        uint256 size;       // how much collateral the challenger provided
        uint256 end;        // the deadline of the challenge (block.timestamp)
        address bidder;     // the address from which the highest bid was made, if any
        uint256 bid;        // the highest bid in ZCHF (total amount, not price per unit)
        ChallengeStatus  status; // record status
    } 

Logic to use challenge status in the business logic


3) Solidity version should be locked for all contracts. Best is to keep the solidity version same for all contracts where possible.
   It is better to lock the contracts to a specific version of Solidity during testing. Using ^0.8.0
   will attempt to run this smart contracts in high version of solidity which are not tested at all.
   
   pragma solidity ^0.8.0;
   Also, is there a reason to using pragma solidity >=0.8.0 <0.9.0 for equity contract.

4) Comparing zero address to make it more reader friendly
   Frankencoin.suggestMinter() function 
   if (minters[_minter] != 0) revert AlreadyRegistered();
  
    to be 
   
   if (minters[_minter] != address(0x0)) revert AlreadyRegistered();

   https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L86
