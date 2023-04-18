## Second check for "challenge.bidder == address(0x0) ? msg.sender " not needed 
In the function end() line 259 ther is a check if challenge.bidder == address(0x0). This check will always be false since one requirement for this part of the code to be reached is in line 254 that require(challenge.challenger != address(0x0)). To save gas on deployment and execution the second check can be removes.

### file: contracts/MintingHub.sol
259:         address recipient = challenge.bidder == address(0x0) ? msg.sender : challenge.bidder; 
-	Change to         address recipient = challenge.bidder;

# Use calldata instead of storage to save gas 
Using calldata for arrays needs less gas than using memory.

### file:  contracts/MintingHub.sol
188:  function minBid(Challenge storage challenge) internal view returns (uint256) {
287: function returnCollateral(Challenge storage challenge, bool postpone) internal {


# Remove function end() in line 235
The function end() in line 235 is for saving gas by making an internal call to the function end() in line 252. But since the function end() in line 252 is not internal but public calling the function end() in line 235 does not save gas but cost more than calling end() in line 252. Therefore to save gas on deployment the function end() in line 235 can be removed. 

### file:  contracts/MintingHub.sol
235: function end(uint256 _challengeNumber) external {
        end(_challengeNumber, false);
    }



# Use a variable for minBid(challenge) instead of calling it twice
The value for minBid(challenge) can be saved in a variable to save gas instead of calling it twice in one function
### file: contracts/MintingHub.sol
216: if (_bidAmountZCHF < minBid(challenge)) revert BidTooLow(_bidAmountZCHF, minBid(challenge))

# Saving gas for the user by failing early
Some checks(if/require) can be moved up in the functions to save gas for the user in case the checks fail. 
### file: contracts/MintingHub.sol
109: require(_initialCollateral >= _minCollateral, "must start with min col")
-	can be moved above line 107 to save gas in case check fails
171: require(challenge.size >= min); 172: require(copy.size >= min);
-	can both be moved above line 167 to save gas for the user in case check fails
216:  if (_bidAmountZCHF < minBid(challenge)) revert BidTooLow(_bidAmountZCHF, minBid(challenge));
-	can be moved to the top of the function to save gas for the user in case it reverts

### file: contracts/Position.sol
81: if (price > _price) revert InsufficientCollateral();
-	setOwner(owner) from line 78 can be moves below this statement to save gas for the user in case the if check fails
