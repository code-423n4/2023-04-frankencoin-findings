## 1. DO NOT PERFORM ARITHMETIC OPERATIONS IN CONSTANTS

Perform the arithmetic operations beforehand and assign the value to the constant to save gas.

    uint256 public constant MIN_HOLDING_DURATION = 90*7200 << BLOCK_TIME_RESOLUTION_BITS; // Set to 5 for local testing

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L59

## 2. USE `unchecked` BLOCK TO SAVE GAS

In the `Frankencoin.equity()` function, the value (balance - minReserve) is returned. But prior to that a check is performed as follows:

      if (balance <= minReserve){
	  
This check makes sure that (balance - minReserve) can never underflow and hence (balance - minReserve) can be unchecked to save gas.

     function equity() public view returns (uint256) {
        uint256 balance = balanceOf(address(reserve));
        uint256 minReserve = minterReserve();
        if (balance <= minReserve){
          return 0;
        } else {
          return balance - minReserve;
        }
      }
	
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L138-L146


## 3. `Equity.canRedeem() external` UTILITY FUNCTION CAN BE REMOVED SINCE `canRedeem(address)` IS A PUBLIC FUNCTION AND CAN BE DIRECTLY CALLED

    function canRedeem() external view returns (bool){
        return canRedeem(msg.sender);
    }
	
No use of the above function since the `canRedeem(address)` can be directly called since it is a public function.

    function canRedeem(address owner) public view returns (bool) {
        return anchorTime() - voteAnchor[owner] >= MIN_HOLDING_DURATION;
    }

This will help to save the deployment gas cost.
	
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L127-L129
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L135-L137

## 4. IN `Equity.votes(address, address[])` CONDUCT THE VARIABLE DECLARATIONS OUTSIDE THE `for` LOOPS TO SAVE GAS

        for (uint i=0; i<helpers.length; i++){
            address current = helpers[i];
			
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L192-L193
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L312-L313

## 5. STATE VARIABLES USED FOR THE SECOND TIME INSIDE THE FUNCTION CAN BE CACHED TO SAVE GAS

   function isMinter(address _minter) override public view returns (bool){
      return minters[_minter] != 0 && block.timestamp >= minters[_minter];
   }

In the `Frankencoin.isMinter()` function, the `minters[_minter]` state variable is accessed twice inside the function.
Hence it can be cached into a memory variable. This memory variable can be used thereafter instead of using the `minters[_minter]` state variable.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L293-L295