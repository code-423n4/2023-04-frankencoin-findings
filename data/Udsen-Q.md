## 1. TRANSACTION WILL REVERT IF THERE IS ANY NON-DELEGATED ADDRESS IN THE `helpers` `address[]` ARRAY, WHEN TRYING TO CALCULATE THE `votes` OF THE DELEGATE

Comment in the `Equity.checkQualified()` states: 

`It is the responsiblity of the caller to figure out whether helpes are necessary and to identify them by scanning the blockchain for Delegation events`

But in the implementation of the `Equity.votes(address, address[])` function, it calls the `Equity.canVoteFor()` function, which makes sure if a `helper` has not delegated to the `owner` directly or indirectly the transaction will revert.

    function canVoteFor(address delegate, address owner) internal view returns (bool) {
        if (owner == delegate){
            return true;
        } else if (owner == address(0x0)){
            return false;
			
Since the `helpers` should be found manually by the user, it is prone to error, and there is a high possiblity that an `user` will include a `non-delegated` helper in the `helper[]` array, which will revert the entire transaction thus wasting gas.

Hence it is recommended to use `if-else` structure instead of `require` and calculate the `votes` of only the `delegated helpers` in the `helpers` address[] array and `ignore` the non-delegated addresses if any, without revert.
Remove the following `require` statement in the `Equity.votes()` function and replace it with a `if-else` statement. 

    require(canVoteFor(sender, current));
	
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L195
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L225-L233

## 2. `<` SHOULD BE REPLACED BY `<=` TO MAKE SURE THERE IS ATLEAST ONE SHARE IN THE `require` STATEMENT OF THE `Equity.calculateProceeds()` FUNCTION.

In `Equity.calculateProceeds()` function there is a `require` statement to check the number of `ZCHF` tokens to redeem.

	require(shares + ONE_DEC18 < totalShares, "too many shares")
	
The comment here states: `make sure there is always at least one share`.

According to the above `require` statement it reverts when there is atleast one share.
And the require statement will only hold `when there is atleast two shares`.
    
Eg: let's assume `totalShares = 1000` and `shares = 999`, so there is atleast one share.
    
But the `require(999 + 1 < 1000)` will be `false` and the transaction will revert. Hence above comment will not stand.

Hence the `require` statement should be updated as below:

    require(shares + ONE_DEC18 <= totalShares, "too many shares")  
	
In this case it will make sure `there is at least one share`.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L293

## 3.  `Equity.redeem` TARGET ADDRESS SHOULD BE CHECKED FOR `address(0)`

In `Equity.redeem` function, when redeeming the pool shares for `ZCHF` tokens, the transfer function is called as follows:

    zchf.transfer(target, proceeds);

But the `target` is never checked for `address(0)`. 
If `address target` is set to `address(0)` erroneously, the proceeds could be sent to `address(0)` and burnt .

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L279

## 4.  ADDRESS `sender` SHOULD BE CHECKED FOR `address(0)` IN THE `equity.votes` FUNCTION.

`Equity.votes(address, address[])` and `Equity.checkQualified()` are public functions. 
Hence if `address(0)` is passed in as `sender` address to these functions, the `Equity.canVoteFor()` will always return `true` when `(owner == delegate)` is checked and both addresses are `address(0)`. 

    function votes(address sender, address[] calldata helpers) public view returns (uint256) {
        uint256 _votes = votes(sender);
        for (uint i=0; i<helpers.length; i++){
            address current = helpers[i];
            require(current != sender);
            require(canVoteFor(sender, current));
			...
			}
	}

    function canVoteFor(address delegate, address owner) internal view returns (bool) {
        if (owner == delegate){
            return true;
        } else if (owner == address(0x0)){
            return false;
		}
		...
	}		

Hence this will erroneously caclulate the votes inside the `Equity.votes()` function when the function actually should revert instead. - Equity.sol#L193 

Current use case of this function is in `Equity.restructureCapTable()` which calls the `checkQualified()` function as below:

            checkQualified(msg.sender, helpers);

Since the msg.sender will not equal to `address(0)`, there is no direct threat. 
But since these are public functions and future changes to the contract could use them for different logic implementations it is recommended to check for the `address(0)`

Or else `if` and `else-if` should interchange in `Equity.canVoteFor()`, so that `address(0)` check will be performed prior to the `owner == delegate` check.
Hence if `owner == address(0)` is checked first then the `Equity.canVoteFor()` will return `false` 

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L193
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L226-L229

## 5. IN `Frankencoin.constructor` THE `_minApplicationPeriod` SHOULD BE CHECKED FOR `0` VALUE (INPUT VALIDATION).

In `Frankencoin.constructor()` function, the immutable variable `MIN_APPLICATION_PERIOD` is set as follows:

      MIN_APPLICATION_PERIOD = _minApplicationPeriod;	
	  
But the `_minApplicationPeriod` is never checked for `0` value. It is recommended to perform input validation since it is being assigned to an immutable variable and can not be changed after assignment.

_minApplicationPeriod should check for `0` value - It is assigned to an immutable variable and hence should be checked for zero value before assigning. - Frankencoin#L60

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L59-L62

## 6. CONSTANTS SHOULD BE DEFINED RATHER THAN USING MAGIC NUMBERS

      return minterReserveE6 / 1000000;
	  
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L118

## 7. CORRECT THE TYPOS FOUND IN THE COMMENTS BELOW: 

Here, `we are also save`, as 68 Bits would imply more than a trillion outstanding shares.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L70

This limit could in theory be reached in times of hyper `inflaction`

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L73

case after their average holding duration is larger than or `equal` the required minimum

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L133

* `helpes` are necessary and to identify - Equity.sol#L207

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L207

* Design rule: Minters calling this method are only allowed to `do` so for tokens amounts they previously minted

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L188

* Under normal circumstances, this is just the `reserver` requirement multiplied by the amount.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L201

Following revert message in the `MintingHub.openPosition()` function can be more descriptive:

     require(_initialCollateral >= _minCollateral, "`must start with min col`");
	 
Should be changed to:

     require(_initialCollateral >= _minCollateral, "`must start with atleast min col`");
	 
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L109

## 8. Use Explicit Variable Declarations FOR DATA TYPES.
    
	use `uint256` instead of `uint`
	
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L192
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L196