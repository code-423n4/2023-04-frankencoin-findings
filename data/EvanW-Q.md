[QA-01] Better to use `SafeTransferFrom` and `SafeTransfer` instead of `TransferFrom` and `Transfer`

below are some examples:

```solidity
contracts/Frankenoin.sol
87: _transfer(msg.sender, address(reserve), _applicationFee);
225: _transfer(address(reserve), payer, assigned); // send reserve to owner
254: _transfer(address(reserve), msg.sender, freedAmount - _amountExcludingReserve); // collect assigned reserve, maybe less than original reserve
283: _transfer(address(reserve), msg.sender, _amount);
285: _transfer(address(reserve), msg.sender, reserveLeft);
```