# Frankencoin

# Suggestion: Use safeTransferFrom

Even thought it’s clearly mentioned [here](https://docs.frankencoin.com/governance/acceptable-collateral)

**The token's transfer function must revert on failure as required by the ERC-20 standard. The Frankencoin system ignores the return value of the transfer method. Tokens that returning "false" to indicate a failure cannot be accepted.**

It might be good to use safeTransferFrom to support non ERC20 compliant tokens as collateral.