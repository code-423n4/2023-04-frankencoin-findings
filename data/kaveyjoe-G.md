Target:contracts/StablecoinBridge.sol

Description:
The horizon variable is set to block.timestamp + 52 weeks, which represents one year from the current block timestamp. However, this may not always be accurate, as the timestamp of the current block can be manipulated by miners to some extent. This means that the actual expiration time of the contract may differ from the intended time. As a result, this bug could allow the contract to be active for a longer period than intended, potentially leading to issues with the stability of the Frankencoin.

Recommendation:
The code should be updated to use the block.number instead of block.timestamp to calculate the horizon, as it is less susceptible to manipulation by miners. The following code can be used instead:

horizon = block.number + (52 weeks / block.timestamp);




