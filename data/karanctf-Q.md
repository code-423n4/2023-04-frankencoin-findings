
## [L-1] `canRedeem` function can be bypassed
[canRedeem](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L135) function in equity.sol can be bypassed if  the paremeter is set to an address that is not insde `voteAnchor` mapping
canRedeem is used in redeem function which returns true if msg.sender is not in voteAnchor
```git
function canRedeem(address owner) public view returns (bool) {
++ require(voteAnchor[owner] > 0);
return anchorTime() - voteAnchor[owner] >= MIN_HOLDING_DURATION;//@audit confusing user
}
```
This will return true but user can't redeem this will confuse user.
Mitigation: Add a check is voteanchor[owner] > 0




## [NC-1] `OnlyOwner` modifier calling another function just to check if` msg.sender` is owner
consider checking it in modifier itself in `ownable.sol`
```git
-- function requireOwner(address sender) internal view {
-- if (owner != sender) revert NotOwner();
-- }//@audit qa and gas not need fo rthis function can be directly used

modifier onlyOwner() {
require(msg.sender == owner);
-- requireOwner(msg.sender);
_;
}

```


## [NC-2] Inconsitency in grouping numbers 
In the code base in multiple places grouping large number like `1000_000` but it should be gouped in thausands which is `1_000_000`
```solidity
uint256 reward = (volume * CHALLENGER_REWARD) / 1000_000;
```

## [NC-3] Check for address 0 in begaining insted of last in permit function
```git
function permit(
address owner,
address spender,
uint256 value,
uint256 deadline,
uint8 v,
bytes32 r,
bytes32 s
) public {

require(deadline >= block.timestamp, "PERMIT_DEADLINE_EXPIRED");
++ require(owner != address(0));
address recoveredAddress = ecrecover(
.
.

);
-- require(recoveredAddress != address(0) && recoveredAddress == owner, "INVALID_SIGNER");//@audit qa check for owner==address(0) in begaining
++ require(recoveredAddress == owner, "INVALID_SIGNER");//@audit qa check for owner==address(0) in begaining
_approve(recoveredAddress, spender, value);
}
}
```
