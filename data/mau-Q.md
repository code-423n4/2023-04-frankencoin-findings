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


