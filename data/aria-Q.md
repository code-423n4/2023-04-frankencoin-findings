# QA report

### Transfer and call function in ERC contract is not used but is vulnerable

**location**: In the Position contract, in the initializeClone function. 

[https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L177) - in line 162.

**explanation:**

The transfer and call function in the ERC-677 contract is not used for a reason, by calling to a function with the transferAndCall, the caller is vulnerable to reentrancy and DOS attacks since the  receiver can control the process with his function and make the transaction as a whole fail on purpose or even to reenter the sender. 
So while this function is not used in the code a use of this function in contract to another is not suggested and dangerous.

In addition, this implementation may encourage security vulnerabilities in the contracts that interact with the coin.

```solidity
// ERC-677 functionality, can be useful for swapping and wrapping tokens
function transferAndCall(address recipient, uint256 amount, bytes calldata data) external override returns (bool) {
	bool success = transfer(recipient, amount);
	if (success){
	    success = IERC677Receiver(recipient).onTokenTransfer(msg.sender, amount, data);
	}
	return success;
}
```

### Shadowing of owner address in the initializeClone function

**location**: In the Position contract, in the initializeClone function. 

[https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol) - in line 76.

**explanation**: 

In the initializeClone function, in the position contract, the contract is using the name â€œownerâ€ for an address. This is shadowing the original â€œownerâ€ which was defined in the Ownable contract. Shadowing values can lead to unexpected results, in this part the wanted value is the one used and therefore itâ€™s not a critical mistake.

```solidity
function initializeClone(address owner, uint256 _price, uint256 _limit, uint256 _coll, uint256 _mint) external onlyHub {
      if(_coll < minimumCollateral) revert InsufficientCollateral();
      setOwner(owner);
      
      price = _mint * ONE_DEC18 / _coll;
      if (price > _price) revert InsufficientCollateral();
      limit = _limit;
      mintInternal(owner, _mint, _coll);

      emit PositionOpened(owner, original, address(zchf), address(collateral), _price);
  }
```

```solidity
contract Ownable {

    address public owner;
	//...
}
```

### big logic error - creating a position with high challenge time:

**location:** In the MintingHub contract in the create position function. 

[https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol) - in line 88.

**explanation:** 

In the contract there is a weird exploit that will let us to create a position in such a way that challenging it will hurt the challenger. In this way we can put the contract in some sort of a hostage situation, when no one wants to challenge the position, yet if the position will pass, it could mint whatever it wants.

The way is by setting the challenging time to sometime far far away, if the challenging time is that big challenging the contract will result in the challengerâ€™s funds being locked. This would also result in the ownerâ€™s funds being locked as well, but still creating a good position that no one will want to challenge is certainly a big threat to the contract and the community. 

This should be fixed by bounding the value in the creation of the position.

### **No immediate incentive for use of the deny function:**

**location:** In the deny functions in the Position contract and the denyMinter in the Frankencoin contract. 

[https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol) - in line 152.

[https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol) - in line 109.

**explanation:** Our deny functions are used by Equity holders to collapse positions and minters, the issue in the whole process is that the caller for the deny function might waste a lot of gas. Now the holders do have an incentive to deny bad positions and minter, but there is no incentive specifically to the sender. This means that while the community as a whole will want to deny, no one specifically will want to deny, since its better for them if others will send. 

This might cause a weird game of chicken between equity holders when each one will want others to deny. The way to solve this problem will probably be by providing some incentive\ gas coverage to the caller to at least try and cover this.    

We propose sending the gas cost of function in it end of it to the msg.sender / transaction.origin in the form of zchf tokens.

NOTE - calculate the delta of the gasleft in the beginning and the end of the funciton so the refund wont be exploitable.