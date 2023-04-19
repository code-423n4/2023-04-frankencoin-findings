1.The method can be set to "external" when only called outside of the function to save gas costs.
   https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L124

2.Blocktime check and balance check should be moved to the head of the function to avoid unnecessary transfers
   https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L51#L52

```solidity
    function mint(address target, uint256 amount) public {
    +   require(block.timestamp <= horizon, "expired");
    +   require(chf.balanceOf(address(this))+amount <= limit, "limit");
        chf.transferFrom(msg.sender, address(this), amount);
        mintInternal(target, amount);
    }

    function mintInternal(address target, uint256 amount) internal {
    -   require(block.timestamp <= horizon, "expired");
    -   require(chf.balanceOf(address(this)) <= limit, "limit");
        zchf.mint(target, amount);
    }
```

3.Putting public immutable values(original,zchf,collateral) in events seems unnecessary
   https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L69
   https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L85

4.Use uint64 instead of uint256 to save more gas,in Solidity, uint64 can represent a maximum of approximately 1.58 billion years in terms of days
   https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L26

   ```solidity
   - uint256 public immutable MIN_APPLICATION_PERIOD; // for example 10 days
   + uint64 public immutable MIN_APPLICATION_PERIOD; // for example 10 days
   ```

5.Use revert custom error instead of require(false)
   https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L81

```solidity
function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool){
    if (msg.sender == address(chf)){
        mintInternal(from, amount);
    } else if (msg.sender == address(zchf)){
        burnInternal(address(this), from, amount);
    } else {
  -     require(false, "unsupported token");
  +     revert customUnsupportedtokenError();
    }
    return true;
}
```

6.Use custom error instead of string to save more gas
   https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L244
   https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L253
   https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L293
   https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L255
   https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L115

7.cache <array> length and move i++ into unchecked{} ,<array>.length Should Not Be Looked Up In Every Loop Of A For-loop

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L190#L202

```solidity
    function votes(address sender, address[] calldata helpers) public view returns (uint256) {
        uint256 _votes = votes(sender);
      + uint256 length = helpers.length;
      - for (uint i=0; i<helpers.length; i++) {
      + for (uint i=0; i<length){
            address current = helpers[i];
            require(current != sender);
            require(canVoteFor(sender, current));
      -     for (uint j=i+1; j<helpers.length; j++){
      +     for (uint j=i+1; j<length){
                require(current != helpers[j]); // ensure helper unique
      +         unchecked {j++;}
            }
            _votes += votes(current);
      +     unchecked {i++;}
        }
        return _votes;
    }
```

8.Time variables use uint32 to save more gas
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L26

```solidity
  -   uint256 public cooldown; // timestamp of the end of the latest cooldown
  +   uint32  public cooldown; // timestamp of the end of the latest cooldown
```