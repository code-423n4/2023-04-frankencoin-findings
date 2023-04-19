 Solidity smart contract for a stablecoin bridge that allows the minting and burning of a Swiss franc stablecoin to and from a Frankencoin. The contract appears to have one issue.




Description:
The check on the limit of outstanding converted source stablecoins seems to be incorrect. The current check verifies whether the balance of source stablecoins held by the contract is less than or equal to the limit. However, this check should be verifying whether the total number of Frankencoins minted by the contract is less than or equal to the limit. As a result, this bug could allow the contract to mint more Frankencoins than intended, potentially leading to issues with the stability of the Frankencoin.

Recommendation:
The code should be updated to check the total number of Frankencoins minted by the contract against the limit instead of the balance of source stablecoins held by the contract. The following code can be used instead:



require(zchf.totalSupply() + amount <= limit, "limit");
