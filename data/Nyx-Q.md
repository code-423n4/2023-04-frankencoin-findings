Low - 1 - Centralization Issue
```
function deny(address[] calldata helpers, string calldata message) public {
        
        if (block.timestamp >= start) revert TooLate();
        IReserve(zchf.reserve()).checkQualified(msg.sender, helpers);
        cooldown = expiration; // since expiration is immutable, we put it under cooldown until the end
        emit PositionDenied(msg.sender, message);
    }
```
Shareholders can prevent opening any position they want.

Low - 2 - burn() function doesnt check horizon

```
function mintInternal(address target, uint256 amount) internal {
        require(block.timestamp <= horizon, "expired");
        require(chf.balanceOf(address(this)) <= limit, "limit");
        zchf.mint(target, amount);
    }
```
The mint() function includes a check to determine whether the bridge is expired or not, but the burn() function does not have a similar check in place.
