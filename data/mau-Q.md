# [N-1] Use a fixed version number in Solidity 

## Comments

It is generally recommended to use a fixed version number in Solidity rather than a version range that includes minor updates.

The reason for this is that minor updates to a Solidity version can introduce changes that affect the behavior of your smart contract. These changes may not necessarily be backward-compatible, which means that your smart contract could break or behave unexpectedly when run on a newer minor version.

By specifying a fixed version number, you ensure that your smart contract is only compiled and deployed with a specific version of Solidity, which reduces the risk of compatibility issues and unexpected behavior.

Additionally, using a fixed version number makes your code more explicit and easier to understand, as it removes any ambiguity around which version of Solidity is being used.

## Instances

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/IERC677Receiver.sol#L2
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L2
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/IFrankencoin.sol#L2
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/IPosition.sol#L2
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/IReserve.sol#L2
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MathUtil.sol#L3
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L2
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L9
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L2
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L2
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L2

# [N-2] Use the latest Solidity version

## Comments

**Security**: Newer versions of Solidity often include security improvements and bug fixes that make the language more secure and less vulnerable to attacks. Using an older version may expose your smart contract to security vulnerabilities that have been addressed in newer versions.

**Compatibility**: Using the latest version of Solidity ensures that your smart contract is compatible with other contracts and tools in the Ethereum ecosystem that are also using the latest version. If you use an older version, you may run into compatibility issues with newer contracts and tools.

**Functionality**: Newer versions of Solidity often introduce new features and functionality that can make your smart contract more powerful and flexible. By using the latest version, you can take advantage of these new features and improve the functionality of your smart contract.

**Support**: Solidity is an open-source project that is actively maintained by a community of developers. By using the latest version, you can access the latest support and resources from the community, which can help you develop and debug your smart contract more effectively.

Overall, using the latest version of Solidity is a good practice to ensure the security, compatibility, functionality, and support of your smart contract. However, it is also important to carefully test and verify any changes that you make to your code to ensure that they work as intended.

## Instances

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/IERC677Receiver.sol#L2
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L2
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/IFrankencoin.sol#L2
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/IPosition.sol#L2
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/IReserve.sol#L2
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MathUtil.sol#L3
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L2
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L9
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L2
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L2
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L2

# [N-3] Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)

It is generally better to use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18) for large numbers. Here are some reasons why:

**Readability**: Scientific notation is more readable than exponentiation, especially when dealing with large numbers. It is easier to quickly recognize the number's magnitude and reduce the likelihood of typos.

**Reduced gas costs**: Using scientific notation can reduce gas costs in contract deployment and transaction execution. This is because Solidity's compiler can optimize scientific notation to use fewer bytes than exponentiation.

**Compatibility**: Scientific notation is compatible with other programming languages, whereas exponentiation may not be available in all programming languages.

Consistency: Using scientific notation promotes consistency and best practices in coding style across different Solidity projects, as it is a widely accepted convention.

## Comments

While the compiler knows to optimize away the exponentiation, it’s still better coding practice to use idioms that do not require compiler optimization, if they exist

## Instances
 
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L25
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MathUtil.sol#L10
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L20

# [N-4] Order of layout

## Comments

According to [Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout), ordering helps readers identify which variable they can use.

Variable should be grouped and ordered by:
- Type declarations
- State variables
- Events
- Modifiers
- Functions

You can move to the top the following instances.

## Instances

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L92
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L130
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L159
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L115
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L231
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L232
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L233
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L366
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L373
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L380
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L387


# [N-5] Split complex require statements into smaller, more specific requirements

## Comments

Splitting complex require statements into smaller, more specific requirements with descriptive reason strings is a best practice in Solidity development, as it helps to improve code readability, maintainability, and security.

## Instances
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L267


# [N-6] Large multiples of ten should use scientific notation (e.g. 1e6) rather than decimal literals (e.g. 1000000), for readability

# Comments

It is generally better to use scientific notation rather than decimal literals for large multiples of ten. 
Here are some reasons why:

**Readability**: Scientific notation is more readable than decimal literals, especially when dealing with large numbers. It is easier to quickly recognize the number's magnitude and reduce the likelihood of typos.

**Clarity**: Scientific notation makes it clear that the number is a multiple of ten, which can be helpful in understanding the context of the code.

**Consistency**: Using scientific notation promotes consistency and best practices in coding style across different Solidity projects, as it is a widely accepted convention.

**Reduced gas costs**: Using scientific notation can reduce gas costs in contract deployment and transaction execution. This is because Solidity's compiler can optimize scientific notation to use fewer bytes than decimal literals.

**Compatibility**: Scientific notation is compatible with other programming languages, whereas decimal literals may not be available in all programming languages.

## Instances 
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MathUtil.sol#L11
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L26

# [L-1] Use ERC20 OpenZeppelin contract

## Comments
The ERC20 contract in OpenZeppelin provides several advantages, including:

**Time-saving**: By using OpenZeppelin's ERC20 implementation, developers can save time and effort as they don't have to start from scratch. OpenZeppelin's ERC20 contract has been audited and tested extensively, which means it is less likely to contain bugs or security vulnerabilities.

**Security**: The OpenZeppelin team has a strong focus on security, and their contracts go through rigorous security audits. Using their ERC20 implementation can help developers avoid security issues that may arise from coding their own implementation from scratch.

**Flexibility**: OpenZeppelin's ERC20 implementation allows developers to customize the token with additional features, such as adding a cap on the total supply, creating a burn function, or implementing a vesting schedule.

**Community support**: OpenZeppelin has a large and active community, which means that developers using their ERC20 implementation can benefit from community support and contributions.

**Upgradability**: OpenZeppelin's ERC20 implementation includes support for upgradability, which means that developers can upgrade their contract in a safe and controlled manner without disrupting the existing token holders.

## Instances

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L41


# [L-2] Constants should be defined rather than using magic numbers

# Comments

Using magic numbers (hardcoded values) in the code makes it harder to understand and maintain. When a magic number is used, it is not immediately clear what the value represents or why it is being used. If the value needs to be changed in the future, you would have to search for all instances of the magic number in the code, which can be time-consuming and error-prone.

By using constants, you can avoid these issues and make your code more robust, readable, and maintainable. Additionally, Solidity provides built-in support for constants, which allows you to define them in a way that optimizes gas usage and makes them easier to work within the context of smart contract development.

# Instances

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L41
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L66
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L189
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L265
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L122
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L124


# [L3]  Change function visibility to external

## Comments

External functions are more restrictive, but they are cheaper to call since they don't need to update the contract's state. Public functions are more flexible but can be more expensive to call, especially when they modify the contract's state.

The function is not being called within the contract, so the external visibility flag can be used instead of public.

## Code

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L91
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L109
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L132

# [L-4]  Use Ownable OpenZeppelin contract

## Comments
The Ownable contract in OpenZeppelin provides several advantages over creating an access control system from scratch, including:

**Security**: OpenZeppelin is a well-audited and tested library of smart contracts, which reduces the risk of vulnerabilities and errors in the code. The Ownable contract provides a secure way to manage access control in your smart contract.

**Time-saving**: The Ownable contract can save a lot of time and effort in writing and testing access control functionalities. Instead of writing your own code, you can use the pre-built contract provided by OpenZeppelin and focus on other aspects of your smart contract.

**Standardization**: OpenZeppelin provides a standard interface for access control, which makes it easier for other developers to understand and interact with your smart contract. By using the Ownable contract, you can benefit from the well-established conventions and best practices in the Ethereum community.

**Flexibility**: The Ownable contract can be customized to fit your specific needs. You can modify the contract to add or remove access control functionalities as per your requirements.

Overall, using the Ownable contract from OpenZeppelin can provide several benefits over creating an access control system from scratch, including security, time-saving, standardization, and flexibility.

## Instances

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L19

# [L-5] Change function visibility to internal

## Comment

It is generally better to use internal visibility instead of external for functions that are only called within the same contract. 

**Reduced gas costs**: External functions have to create a message call to be invoked, which incurs additional gas costs. Internal functions, on the other hand, can be called directly within the same contract, without the need for a message call, resulting in reduced gas costs.

**Better security**: Using internal visibility for functions that are only called within the same contract can improve the security of the contract. This is because external functions are callable from outside the contract, which can create potential attack vectors if they are not properly secured.

**Easier to reason about**: Using internal visibility can make the contract code easier to read and reason about, as it clearly indicates that the function is only used internally within the contract.

**Consistency**: Using internal visibility for internal functions promotes consistency and best practices in coding style across different Solidity projects, as it is a widely accepted convention.

## Instances
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L159
