A business logic that in certain circumstances damages the value of this project:
In the logic of this project, it is assumed that this stable coin (ZCHF) can be exchanged one-to-one with other stable coins with a similar base currency, or they can be pledged and its equivalent (ZCHF) will be minted.
Since no type of oracle is involved in this project, in the event that the value of another stable coin falls, users start minting in this project by pledging it, and in this way, they transfer the same risk of falling value to this project.

========================================================================================

The following statement is written in the project document:
“. There are two ways to initiate a position: one can either create a completely new one with arbitrary parameters, or one can clone an existing position. The former is for advanced users and not exposed in the frontend. The latter is the faster way of obtaining Frankencoin against a collateral and supported in the frontend.”


It seems that not putting the first method in the front end creates a special advantage for a number of users who have the ability and technical knowledge to use it. In addition to reducing the number of public users of the project, this issue creates a kind of injustice in accessing opportunities in this project (it is clear that everyone has the same opportunity for both modes, but in practical use, certain people have the ability to use the first mode)

========================================================================================

The following statement is written in the project document:

“Note that a fraction of what you borrow does not go to your wallet, but instead is sent to the borrowers reserve in your name. Unless the Frankencoin system suffers from large losses in the meantime, you will get that reserve back as you repay the outstanding amount”.

The downside to this method is that if there is a huge loss, the coins reserved for compensation will lose a lot of their value. And they do not have the ability to compensate for the damage caused. 
It is suggested that the value considered for the reserve should be in the form of an asset of another type and less risky (for example, it should be a more stable cryptocurrency or even in the form of fiat).

========================================================================================

A mechanism for checking the conditions mentioned in the following document is not considered:

Acceptable Collateral:
https://docs.frankencoin.com/governance/acceptable-collateral

========================================================================================

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L39

-Checking the zero address is not considered and because the change of ownership is done in one-step sending wrong or zero address has very bad consequences.

========================================================================================



https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L89

Checking the zero address is not considered
========================================================================================

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L143


https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L174 

In both lines of code above, the return value is stored before the changes are made, and this makes the return value different from the actual length of the array.
It is recommended to assign the value to “pos”   just before the return statement
========================================================================================

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L156

  Each user can run this function over and over again for a challenge, thus splitting a good challenge into a large number of smaller challenges. This may ruin the chances of a good challenge

Solution: There needs to be a limit to the number of times a main challenge can be scaled down

========================================================================================


https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf6

2543f68f/contracts/Frankencoin.sol#L25

 MIN_FEE = 1000 * (10**18);

This expression can be calculated, and its constant value can be written in the code, and as a result, a small amount of gas is reduced.
========================================================================================


https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L45


According to the value of the epoch time number in the real world, uint64 or even smaller data type can be used instead of uint256 and save a lot of memory (128 bytes or even more per minter).

========================================================================================

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L83


Smaller variables can be used for period and fee input arguments

========================================================================================

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L165



That the minter himself can choose fee, so he/she can manipulate the amount of usableMint value.
And this issue causes inconsistency in the percentage of mints and payments.
It is suggested to consider fee as fixed or at least in a certain range.


