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

