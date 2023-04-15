**contracts/Position.sol**
- L138 - When the owner uses the adjust() function and defines the newCollateral, it is confusing that it is not clear how much is going to be transferred in the transferFrom, this can lead to several reverts because not knowing how much is going to be transferred specifically ( without consulting another function) I do not perform the corresponding approve, so perhaps it would be less confusing if the input reports how much it wants to add to the balance and use an event to report the current state of the balance.

- L132 - The adjust() function is responsible for updating the collateral and updating the Frankencoin tokens, therefore it is performing two tasks at the same time, something that does not follow good practices when writing code.
It is recommended that there be two functions and that each one only handles one process.


**contracts/MintingHub.sol**
- L37 - There is commented code that should be removed as it can cause confusion.

- L88/124 - Lack of event emission after critical functions as openPosition() and clonePosition().


**contracts/StablecoinBridge.sol**
- L81 - To perform a revert, a require(false) is used, this is an incorrect use of require, the correct thing to do would be to use this: revert("unsupported token");

- L75 - The onTokenTransfer() function returns a bool and this is not very useful in this case, since the bool does not help us to know what was executed, nor what was the result, therefore the most useful thing would be for the function to be void and have the mintInternal() and burnInternal() functions raise events.


**contracts/ERC20PermitLight.sol**
- L35 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.

- L65 - There is a commented line of code, this reduces the understanding of the code, therefore it should be removed.


**contracts/Frankencoin.sol**
- L266 - The name of the modifier is minterOnly() this is somewhat confusing the correct name should be onlyMinter().
