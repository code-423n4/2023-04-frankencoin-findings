1) MintingHub contract's function openPosition()
   The below condition is checked in the overloaded function that created the position.
   Since the condition is based on input parameter, this check could be done much early in the flow of 
   the program.
   
   Idealy, create a modifier with the below require condition and attach to both openPosition functions.

   require(_initialCollateral >= _minCollateral, "must start with min col");