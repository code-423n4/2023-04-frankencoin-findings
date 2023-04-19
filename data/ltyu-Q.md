## [L-1] `isClosed` can be incorrect
As noted in the code documentation for the function `isClosed`:
```
A position should only be considered 'closed', once its collateral has been withdrawn.
* This is also a good creterion when deciding whether it should be shown in a frontend.
```

Currently, `isClosed()` relies on the balance of the collateral in the contract. 
```
function collateralBalance() internal view returns (uint256){
    return IERC20(collateral).balanceOf(address(this));
}
```
This is problematic because collateral can be sent to the “closed” Position to make it seem like the position is still open. This can throw off the front-end or any other system that relies on this function.