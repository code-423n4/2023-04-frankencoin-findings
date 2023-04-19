# QA and Low

### 1. Simplify isMinter check in Frankencoin 

The check `minters[_minter] != 0` is not necessary. if the `minters` timestamp is set, it will be greater 0. 

```solidity
   /**
    * Returns true if the address is an approved minter.
    */
   function isMinter(address _minter) override public view returns (bool){
      return minters[_minter] != 0 && block.timestamp >= minters[_minter];
   }

```

### 2. Lack of using events
There is a general lack of events in the codebase.

```diff
    
   function registerPosition(address _position) override external {
      if (!isMinter(msg.sender)) revert NotMinter();
      positions[_position] = msg.sender;
+     emit NewPosition(position, msg.sender);      
   }

```

### 3. Error definitions are not defined in contract header
A standard pattern in Solidity is to define the storage variables and events at the beginning of the contract.

In Frankencoin some defintions are not at the beginning like
```
// Frankencoin
Error `NotMinter` is defined between functions
```
### 1. Simplify logical reverts with require in the codebase
A common pattern in the Frankencoin codebase is the following:
```solidity
if (!logicalCondition) revert;
```
However, this `if not` revert are more complicated to read then.

```solidity
require(logicalCondition);
```

**Example from Frankencoin.sol**

```solidity
   modifier minterOnly() {
      if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();
      _;

   }
```

**Cleaner Way**
```solidity
modifier minterOnly() {
    require(isMinter(msg.sender) || isMinter(positions[msg.sender]), "NotMinter");
    _;
}
```
The revert pattern with custom error types is cheaper from the gas perspective but makes the logical conditions harder to understand.