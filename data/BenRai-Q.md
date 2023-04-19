# Follow convention for the order of events, errors, modifiers and Interfaces
Errors, events, modifiers and interfaces in the following contracts are scattered all over the code and/or sit below the functions. To meet the convention and have a better overview they should be grouped together and put above the functions of the contract. The affected contracts are:

Frankencoin.sol
MintingHub..sol
Position.sol



# There some typos or missing words in comments

### file: contracts/ MintingHub.sol
258: // notify the position that will send the collateral to the bidder
Should be: notify the position that the collateral will be send to the bidder

### file: contracts:/ Position.sol
358:      * This is also a good creterion when deciding whether it should be shown in a frontend.
Criterion should be Critirion

### file: contracts/MintingHub.sol
153:      * have the liquidity available to bid a sufficient amount. With this function, the can split of smaller slices of 
Should be: With this function, the position owner can split / or “anyone can split”

### file: Frankencoin.sol
56:     * Initiates the Frankencoin with the provided minimum application period for new plugins
Comment should not say plugins but minters for better understanding
188: * Design rule: Minters calling this method are only allowed to so for tokens amounts they previously minted with the same _reservePPM amount.

Should be: … only allowed to *do* so for…



# Event Name should be PostponedReturn() with only one capital P
### file: contracts/MintingHub.sol
52:     event PostPonedReturn(address collateral, address indexed beneficiary, uint256 amount);
# Comment for example should not be in the code but above the code

To be consistent with the other comments the comments for the example should be above the code not within the code of the function.

### fiel: contracts/Frankencoin.sol
 lines 235 - 240

# Remove TODOs from comments

### file: contracts/MintingHub.sol
74:      * TODO: in future versions, it might be better to fix the interest and not the fee

# transfareAndCall() should not be “override” since it is not inherited from ERC20

transfareAndCall() is no inhareded from IERC20 so it should no have the keyword “override” in it

file: contract/ERC20.sol
162:     function transferAndCall(address recipient, uint256 amount, bytes calldata data) external override returns (bool) {

#  Comments are not consistent with the code
In the comment in line 36-38 two function are mentioned that do not exist in the contract. For clarity this lines should be removed since there are no `decreaseAllowance` and `increaseAllowance` functions in the contract.

### file: contracts/ERC20.sol
36-38: * Finally, the non-standard `decreaseAllowance` and `increaseAllowance`
 * functions have been added to mitigate the well-known issues around setting
 * allowances. See `IERC20.approve`.

# Function ` restructurCapital()`  can run out of gas 

If the address array for the address to wipe gets to long the function might run out of gas and therefore restructuring the capital might not be possible.
File: equity.sol
312:         for (uint256 i = 0; i<addressesToWipe.length; i++){ 

