L-01:onTokenTransfer() Event Trade Parameter `who` be set incorrectly

in `onTokenTransfer` When emitting events, the wrong `who=msg.sender` is set, and `who=from` should be passed in.

```solidity
function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool) {
        require(msg.sender == address(zchf), "caller must be zchf");
...
-       emit Trade(msg.sender, int(shares), amount, price());
+       emit Trade(from, int(shares), amount, price());
```

L-02:denyMinter() when `block.timestamp == minters[_minter]` can no longer execute denyMinter()
1. Currently `denyMinter()`, when `block.timestamp == minters[_minter]` can be deny
2. But in the `isMinter()` method `block.timestamp == minters[_minter]` is already `minter`.

So when `block.timestamp == minters[_minter]`,`denyMinter()`, it should refuse to deny

```solidity
   function denyMinter(address _minter, address[] calldata _helpers, string calldata _message) override external {
-     if (block.timestamp > minters[_minter]) revert TooLate();
+     if (block.timestamp >= minters[_minter]) revert TooLate();
      reserve.checkQualified(msg.sender, _helpers);
      delete minters[_minter];
      emit MinterDenied(_minter, _message);
   }
```

L-03:createClone() Need to add the judgment of the created contract address! =address(0) to avoid creating a failed contract and continue execution

```solidity
    function createClone(address target) internal returns (address result) {
        bytes20 targetBytes = bytes20(target);
        assembly {
            let clone := mload(0x40)
            mstore(clone, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
            mstore(add(clone, 0x14), targetBytes)
            mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
            result := create(0, clone, 0x37)
        }

+       require(result != address(0), "ERC1167: create failed");
    }
```
