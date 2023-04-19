### [Low-01] User May Call Wrong Function.
According to their Docs ```For now, it is not possible to open new positions through the frontend. But bold degens can do so on Etherscan or through their other tool of choice. ```

It clearl signifies that to open up a New Position User will intaract with code base, Etherscan.
Where a User can by mistakely call ```createNewPosition()``` of ```PositionFactory.sol``` instead of calling ```openPosition()``` from ```MintingHub.sol```
As a result his position get created but it will not be registered as those are preformed inside ```openPosition()```

This will create a confusion as User transaction get successful even if his position not registered and without fund transfer. This may harm user experience.

*Mitigation*
Its clear that Creating a Position always initiated from ```MintingHub.sol``` contract, this will forther call ```createNewPosition()``` of ```PositionFactory.sol```. So if any user directly call ```createNewPosition()``` transaction should revert.

Basically ```createNewPosition()``` should have a check(```require()``` statement) that ensure call come from ```MintingHub.sol``` contract

```solidity
    function createNewPosition(address _owner, address _zchf, address _collateral, uint256 _minCollateral, 
        uint256 _initialLimit, uint256 initPeriod, uint256 _duration, uint256 _challengePeriod, 
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reserve) 
+       external callFromHub returns (address) {
        return address(new Position(_owner, msg.sender, _zchf, _collateral, 
            _minCollateral, _initialLimit, initPeriod, _duration, 
            _challengePeriod, _mintingFeePPM, _liqPrice, _reserve));
    }
+   modifier callFromHub(){
+           require(msg.sender == HUB);
+           _;
+   }
```
*Instances(1)*
```
File:: contracts/PositionFactory.sol
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L13-L20
```

### [Low-02] Instead Of Using ```CREATE``` Should Consider To Use ```CREATE2```
EVM there are two opcodes to make a ```create``` and ```create2``` smart contract from another smart contract. 

The differences between the CREATE and CREATE2 opcodes:

An important difference lies in how the address of the new contract is determined.

With ```CREATE``` the address is determined by the factory contract's ```nonce```. Everytime CREATE is called in the factory, its nonce is increased by 1.

This approach is very controversial and the recent hack with Optimism was just related to this. https://rekt.news/wintermute-rekt/

With ```CREATE2```, the address is determined by an arbitrary salt value and the init_code.

The big advantage of CREATE2 is that the destination address is not dependent on the exact state (i.e. the nonce) of the factory when it's called. This allows transaction results to be simulated off-chain, which is an important part of many state channel based approaches to scaling.

```solidity
function createClone(address target) internal returns (address result) {
        bytes20 targetBytes = bytes20(target);
        assembly {
            let clone := mload(0x40)
            mstore(clone, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
            mstore(add(clone, 0x14), targetBytes)
            mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
            result := create(0, clone, 0x37) // @audit-issue
        }
   }
```
*Instances(1)*
```
File:: contracts/PositionFactory.sol
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol#L37-L45
```

### [Low-03] Minter Can Goes Malicious And Can Mint Infinite Amount Of FrankenCoin(ZCHF)
```solidity
   function mint(address _target, uint256 _amount) override external minterOnly {
      _mint(_target, _amount);
   }
```
```mint``` is a ```minterOnly``` function 
and anyone can become minter via (if got accepted)
```solidity
   function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {
      if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort(); 
      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
      if (minters[_minter] != 0) revert AlreadyRegistered();
      _transfer(msg.sender, address(reserve), _applicationFee);
      minters[_minter] = block.timestamp + _applicationPeriod;
      emit MinterApplied(_minter, _applicationPeriod, _applicationFee, _message); 
   }
```

After becoming a minter he can mint as much as token he can.
There is no method present to remove malicious actor.

So its a single point of failure.
*Instances(1)*
```
File:: contracts/Frankencoin.sol
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L83-L89
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L125-L128
```


### [Low-04] Modifier ```minterOnly``` Implemented Wrongly
```solidity
   modifier minterOnly() {
      if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter(); 
      _;
   }
``` 
Here problem is in ```!isMinter(positions[msg.sender])```
Mapping Position ```positions[msg.sender]``` return a address of Position.

```solidity
  function isMinter(address _minter) override public view returns (bool){
      return minters[_minter] != 0 && block.timestamp >= minters[_minter];
   }
```

so that's why ```!isMinter(positions[msg.sender])``` always return true, what ever condition may be.

So second step in `if` clause is completely useless so remove it.

*Instances(1)*
```
File:: contracts/Frankencoin.sol
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L293-L295
```