# The withdraw function will fail when extracting non-standard ERC20 tokens.

Position.sol L253
```
    /**
     * Withdraw any ERC20 token that might have ended up on this address.
     * Withdrawing collateral is subject to the same restrictions as withdrawCollateral(...).
     */
    function withdraw(address token, address target, uint256 amount) external onlyOwner {
        if (token == address(collateral)){
            withdrawCollateral(target, amount);
        } else {
            IERC20(token).transfer(target, amount);
        }
    }
```

Although the document states that the collateral needs to be reverted in case of a failed transfer, this function also has the ability to extract non-collateral tokens, which requires `safeTransfer` to ensure availability.

