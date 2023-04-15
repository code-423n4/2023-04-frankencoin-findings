## Infinite token allowance
The smart contract supports "infinite allowance", which means that when one account grants another account an allowance, if the amount is (2^255) or above, it represents that there are no limits on the use of the tokens for that account, and the account can transfer the tokens an infinite number of times. While this design facilitates the use of tokens, it also poses potential security risks because it means that one account can have perpetual transfer permission for another account's tokens. It should be noted that this design does not conform to the ERC-20 standard or other common token standards.
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol
## Transfer failure handling
When a user invokes the transfer() or transferFrom() function, if the balance or token authorization is insufficient, it will trigger an internal error and throw a custom exception. Although this approach conforms to the Solidity function usage specification, it may make it difficult for external callers to determine the specific error reason, thus increasing the difficulty of error handling in the application.
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol
## Suggest adding checks for blacklisted addresses
The smart contract does not implement checks and handling for blacklisted addresses, which may lead to the abuse of tokens by malicious users. Therefore, it is recommended to add checks and handling for blacklisted addresses in practice.
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol
## Lack of batch transfer functions:

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol
The smart contract does not implement batch transfer functions, such as transferBatch(address[] recipients, uint256[] amounts), etc. The lack of these functions will increase the transaction costs and time significantly, while reducing the usability of the smart contract.
## Suggest adding prevention mechanisms for reentrancy attacks
Although this smart contract does not directly call other smart contracts, external calls may be involved in future feature expansion. Therefore, to prevent reentrancy attacks, it is recommended to add reentrancy guard mechanisms before function execution.
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol

## Using create() function to create a child contract
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L13

This smart contract creates a new child contract by calling the create() function. It should be noted that when using the create() function, if malicious EVM code is accidentally passed, it may cause potential security vulnerabilities in the child contract and jeopardize the overall system's security. Therefore, in practical use, it is necessary to ensure that the code passed to the create() function is trustworthy and secure.

## Signature verification issue
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol
The ecrecover function is used in this contract to verify signatures. If the signature is invalid, it will lead to authorization failure. However, the ecrecover function itself is vulnerable to attacks, for example, by constructing malicious data to deceive the signature verification process. Therefore, in practical use, more stringent signature verification is required to avoid such attacks.
## Preventing replay attacks

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol
 the permit function uses the nonce variable to prevent replay attacks. Still, the update method of the nonce variable is unchecked, which may lead to nonce variable overflow and reuse, thereby increasing the risk of attack.




