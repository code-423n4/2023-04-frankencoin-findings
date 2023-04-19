# 1
## the order of the contract layout should follow the recommended layout as specifiied in the solidity documentation.
some of the layout of the contracts in the frankencoin project were not structured according to the format specified in solidity documentation. A contract in the specified format is considered best practice to enhance code readability. check [here](https://docs.soliditylang.org/en/v0.8.19/style-guide.html#order-of-layout) for more details.
the format of a contract should follow this order:
 Type declarations
 State variables
 Events
 Errors
 Modifiers
 Functions
# context. the layout in the following contract do not follow the recommended format specified in the documentation.
1) [position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol)
2) [minting hub] (https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol)
3) [Equity](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol)
4) [frankencoin] (https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol)
5) [https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol]

If you look at these contracts stated above properly, their modifier is place at the end of all functions and their error message were placed in between the functions. ( thou some were mentioned in the automated findings but not all were regerenced like the modifier and error message stated here)

# 2.
## Variable overshadow.
The variable Owner declare [here](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L76) overshadowed the state variable declared in the Ownable.sol [here](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L21) which can be misleading.

# 3. The protocol should pay maximum attention to the on chain transaction happening at every point in time.
This is a warning to the protocol to pay attention to every action happening on the blockchain especially with the function `OpenPosition` which is not implemented on the frontend. thou there is a minimum delay of 3 days. A malicious user can deploy a non valuable token and use it as a collateral and if such position is not denied within 3 days of it's opening it can lead to an abitrary miniting of frankentoken by the attacker and nobody will be able to challenge such as he is the only holder of such collateral. An attacker can also  open different position with a malicious position in between, if the protocol got carried away without carefully checking all the position this can lead to the attacker getting away with his intention.

proper scanning, check  and security should be given to all the position opening at every point in time because a slight mistake can lead to a total loss for the protocol.