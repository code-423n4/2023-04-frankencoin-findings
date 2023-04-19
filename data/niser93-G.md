# GAS REPORT

## GAS-01 - Use alternative formulation in order to avoid multiplication

### Founded in [MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol)

* [MintingHub.sol#L189](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L189)

```
return (challenge.bid *  1005) /  1000;
```

### Description
Another way to increase a value X by 5‰ is:

```
X * 1.005      is equal to
X * 1005 / 1000      is equal to
X * (1000/1000) + X * (5/1000)     is equal to
X + X/200
```
However, working with uint, we must be care about rounding.

### Proof of concept

Let call F, float result. It is in this form: a.b,
where 
* a is integer part
* b is floating part

We can have 3 cases:
* 0 < b < 0.5
* b == 0.5
* b > 0.5

So, due to the fact that in both formulas we apply only one division operation,
we can show equality of formulas using 3 example for this 3 cases:


Case 1 (0 < b < 0.5):
```
challenge.bid = 1037

(challenge.bid *  1005) /  1000 = 1.042,185
return 1.042

challenge.bid + challenge.bid/200 = 1037 + 5,185 = 1037 + 5 
return 1.042
```
Case 2 (b == 0.5):
```
challenge.bid = 1100

(challenge.bid *  1005) /  1000 = 1.105,5
return 1.105

challenge.bid + challenge.bid/200 = 1100 + 5,5 = 1100 + 5 
return 1.105
```
Case 3 (b > 0.5):
```
challenge.bid = 1111

(challenge.bid *  1005) /  1000 = 1.116,555
return 1.116

challenge.bid + challenge.bid/200 = 1111 + 5,555 = 1111 + 5 
return 1.116
```

### Test with customized contracts

Using Remix IDE with optimization at 10k runs.

```
contract attempt{
	function tryGas(uint256 ch)  public  pure  returns(uint256){
		return  (ch *  1005)  /  1000;
	}
}

Execution cost: 409 
```

```
contract attempt{
	function tryGas(uint256 ch)  public  pure  returns(uint256){
		return  (ch +  (ch/200));
	}
}

Execution cost: 387
```



## GAS-02 - Consider to use require instead of revert + error

### Founded in [Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol)

* [Position.sol#L77](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L77)

```
if(_coll < minimumCollateral) revert  InsufficientCollateral();
```

* [Position.sol#L81](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L81)

```
if (price > _price) revert  InsufficientCollateral();
```

* [Position.sol#L110](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L110)

```
if (block.timestamp  >= start) revert  TooLate();
```

* [Position.sol#L194](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L194)

```
if (minted + amount > limit) revert  LimitExceeded();
```

* [Position.sol#L240](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L240)

```
if (amount > minted) revert  RepaidTooMuch(amount - minted);
```
In this case, RepaidTooMuch is used:
```
error RepaidTooMuch(uint256  excess);
```
However, uint256 excess is not used.

* [Position.sol#L283](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L283)

```
if (collateralReserve * atPrice < minted * ONE_DEC18) revert  InsufficientCollateral();
```

* [Position.sol#L294](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L294)

```
if (size < minimumCollateral && size <  collateralBalance()) revert  ChallengeTooSmall();
```

* [Position.sol#L364-L390](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L364-L390)

```
error Expired();
modifier alive() {
	if (block.timestamp > expiration) revert Expired();
	_;
}

error Hot();
modifier noCooldown() {
	if (block.timestamp <= cooldown) revert Hot();
	_;
}

error Challenged();
modifier noChallenge() {
	if (challengedAmount > 0) revert Challenged();
	_;
}

error NotHub();
modifier onlyHub() {
	if (msg.sender != address(hub)) revert NotHub();
	_;
}
```

### Founded in [MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol)

In which error are defined after their usage.

* [MintingHub.sol#L201](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L201)

```
if (block.timestamp  >= challenge.end) revert  TooLate();
```

* [MintingHub.sol#L202](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L202)

```
if (expectedSize != challenge.size) revert  UnexpectedSize();
```

* [MintingHub.sol#L216](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L216)

```
if (_bidAmountZCHF <  minBid(challenge)) revert  BidTooLow(_bidAmountZCHF, minBid(challenge));
```
In this case, BidTooLow is used:
```
error BidTooLow(uint256  bid, uint256  min);
```
However, uint256 bid and uin256 min are not used (and bid has same name of a function).

### Founded in [Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol)

In which "error" are defined after their usage.

* [Equity.sol#L211](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L211)

```
if (_votes *  10000  < QUORUM *  totalVotes()) revert  NotQualified();
```

### Founded in [Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol)

In which "error" are defined after their usage.

* [Frankencoin.sol#L84](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84)

```
if (_applicationPeriod < MIN_APPLICATION_PERIOD &&  totalSupply() >  0) revert  PeriodTooShort();
```

* [Frankencoin.sol#L85](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L85)

```
if (_applicationFee < MIN_FEE &&  totalSupply() >  0) revert  FeeTooLow();
```

* [Frankencoin.sol#L86](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L86)

```
if (minters[_minter] !=  0) revert  AlreadyRegistered();
```

* [Frankencoin.sol#L126](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L126)

```
if (!isMinter(msg.sender)) revert  NotMinter();
```

* [Frankencoin.sol#L153](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L153)

```
if (block.timestamp  > minters[_minter]) revert  TooLate();
```

* [Frankencoin.sol#L267](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L267)

```
if (!isMinter(msg.sender) &&  !isMinter(positions[msg.sender])) revert  NotMinter();
```

### Description

Using 
```
require(!condition)
```
saves gas with respect using 
```
if(condition) revert error();
```

Take care that 
```
require(!condition, "error description")
```
cost more gas. 
So you could save gas only in case you don't need error description.

### Proof of concept with customized contracts

Using Remix IDE with optimization at 10k runs.

```
contract attempt{
	error InsufficientCollateral();
	function tryGas(uint256 _coll,  uint256 minimumCollateral)  public  pure  {
		if(_coll < minimumCollateral)  revert InsufficientCollateral();
	}
}

Execution cost: 283 
```

```
contract attempt{
	error InsufficientCollateral();
	function tryGas(uint256 _coll,  uint256 minimumCollateral)  public  pure  {
		require(_coll >= minimumCollateral);
	}
}

Execution cost: 244
```


## GAS-03 - Using == instead of != 

### Founded in [Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol)

* [Equity.sol#L157-L167](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L157-L167)

```
	function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){
	if (to != address(0x0)) {
		uint256 recipientVotes = votes(to); // for example 21 if 7 shares were held for 3 blocks
		uint256 newbalance = balanceOf(to) + amount; // for example 11 if 4 shares are added
		voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); // new example anchor is only 21 / 11 = 1 block in the past
		return recipientVotes % newbalance; // we have lost 21 % 11 = 10 votes
	} else {
		// optimization for burn, vote anchor of null address does not matter
		return 0;
	}
}
```


### Description
Using == instead of != saves gas.

```
if (to != address(0x0)) {
	uint256 recipientVotes = votes(to); // for example 21 if 7 shares were held for 3 blocks
	uint256 newbalance = balanceOf(to) + amount; // for example 11 if 4 shares are added
	voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); // new example anchor is only 21 / 11 = 1 block in the past
	return recipientVotes % newbalance; // we have lost 21 % 11 = 10 votes
} else {
	// optimization for burn, vote anchor of null address does not matter
	return 0;
}
```

becames

```
if (to == address(0x0)) {
		// optimization for burn, vote anchor of null address does not matter
	return 0;
} else {
	uint256 recipientVotes = votes(to); // for example 21 if 7 shares were held for 3 blocks
	uint256 newbalance = balanceOf(to) + amount; // for example 11 if 4 shares are added
	voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); // new example anchor is only 21 / 11 = 1 block in the past
	return recipientVotes % newbalance; // we have lost 21 % 11 = 10 votes
}
```

You could also avoid return 0.

I sent other gas optimization report for that.


### Proof of concept with customized contracts

Using Remix IDE with optimization at 10k runs.

```
contract attempt{
	function tryGas(address to)  public  pure  returns(uint256){
		if  (to !=  address(0x0))  {
			return  1;
		}  else  {
			return  0;
		}
	}
}

Execution cost: 319 (with "address to" non-zero address)
Execution cost: 320 (with "address to" == 0x0)
```

```
contract attempt{
	function tryGas(address to)  public  pure  returns(uint256){
		if  (to ==  address(0x0))  {
			return  0;
		}  else  {
			return  1;
		}
	}
}

Execution cost: 317 (with "address to" non-zero address)
Execution cost: 316 (with "address to" == 0x0)
```

## GAS-04 - Let return 0 instead of explicity write it

### Founded in [Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol)

* [Frankencoin.sol#L102-L111](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L102-L111)

```
function allowanceInternal(address owner, address spender) internal view override returns (uint256) {
	uint256 explicit = super.allowanceInternal(owner, spender);
	if (explicit > 0){
		return explicit; // don't waste gas checking minter
	} else if (isMinter(spender) || isMinter(isPosition(spender))){
		return INFINITY;
	} else {
		return 0;
	}
}
```

### Founded in [Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol)

* [Frankencoin.sol#L138-L146](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L138-L146)

```
function equity() public view returns (uint256) {
	uint256 balance = balanceOf(address(reserve));
	uint256 minReserve = minterReserve();
	if (balance <= minReserve){
		return 0;
	} else {
		return balance - minReserve;
	}
}
```

### Founded in [Equity.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol)

* [Equity.sol#L157-L167](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L157-L167)

```
	function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){
	if (to != address(0x0)) {
		uint256 recipientVotes = votes(to); // for example 21 if 7 shares were held for 3 blocks
		uint256 newbalance = balanceOf(to) + amount; // for example 11 if 4 shares are added
		voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); // new example anchor is only 21 / 11 = 1 block in the past
		return recipientVotes % newbalance; // we have lost 21 % 11 = 10 votes
	} else {
		// optimization for burn, vote anchor of null address does not matter
		return 0;
	}
}
```


### Description
"returns (uint256)" will return 0, if no other return is triggered.

* [Frankencoin.sol#L102-L111](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L102-L111)

```
function allowanceInternal(address owner, address spender) internal view override returns (uint256) {
	uint256 explicit = super.allowanceInternal(owner, spender);
	if (explicit > 0){
		return explicit; // don't waste gas checking minter
	} else if (isMinter(spender) || isMinter(isPosition(spender))){
		return INFINITY;
	} else {
		return 0;
	}
}
```

becames

```
function allowanceInternal(address owner, address spender) internal view override returns (uint256) {
	uint256 explicit = super.allowanceInternal(owner, spender);
	if (explicit > 0){
		return explicit; // don't waste gas checking minter
	} else if (isMinter(spender) || isMinter(isPosition(spender))){
		return INFINITY;
	}
}
```


* [Frankencoin.sol#L138-L146](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L138-L146)

```
function equity() public view returns (uint256) {
	uint256 balance = balanceOf(address(reserve));
	uint256 minReserve = minterReserve();
	if (balance <= minReserve){
		return 0;
	} else {
		return balance - minReserve;
	}
}
```

becames 

```
function equity() public view returns (uint256) {
	uint256 balance = balanceOf(address(reserve));
	uint256 minReserve = minterReserve();
	if(balance >= minReserve) return balance - minReserve;
}
```

* [Equity.sol#L157-L167](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L157-L167)

```
	function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){
	if (to != address(0x0)) {
		uint256 recipientVotes = votes(to); // for example 21 if 7 shares were held for 3 blocks
		uint256 newbalance = balanceOf(to) + amount; // for example 11 if 4 shares are added
		voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); // new example anchor is only 21 / 11 = 1 block in the past
		return recipientVotes % newbalance; // we have lost 21 % 11 = 10 votes
	} else {
		// optimization for burn, vote anchor of null address does not matter
		return 0;
	}
}
```

becames

```
	function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){
	if (to != address(0x0)) {
		uint256 recipientVotes = votes(to); // for example 21 if 7 shares were held for 3 blocks
		uint256 newbalance = balanceOf(to) + amount; // for example 11 if 4 shares are added
		voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); // new example anchor is only 21 / 11 = 1 block in the past
		return recipientVotes % newbalance; // we have lost 21 % 11 = 10 votes
	}
}
```

### Proof of concept with customized contracts

Using Remix IDE with optimization at 10k runs.

```
contract attempt{
	function tryGas(address to)  public  pure  returns(uint256){
		if  (to !=  address(0x0))  {
			return  1;
		}  else  {
			return  0;
		}
	}
}

Execution cost: 319 (with "address to" non-zero address)
Execution cost: 320 (with "address to" == 0x0)
```

```
contract attempt{
	function tryGas(address to)  public  pure  returns(uint256){
		if  (to !=  address(0x0))  {
			return  1;
		}
	}
}

Execution cost: 319 (with "address to" non-zero address)
Execution cost: 315 (with "address to" == 0x0)
```



## GAS-05 - Consider to modify OwnershipTransferred event in order to avoid oldOwner variable and save gas

### Founded in [Ownable.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol)

* [Ownable.sol#L39-L43](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L39-L43)

```
function setOwner(address newOwner) internal {
	address oldOwner = owner;
	owner = newOwner;
	emit OwnershipTransferred(oldOwner, newOwner);
}
```


### Description
Modify OwnershipTransferred event in OwnershipTransferredTo event, which only indicated newOwner.
Then, avoid to create placeholder variable oldOwner in order to save gas

```
event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
[...]
function setOwner(address newOwner) internal {
	address oldOwner = owner;
	owner = newOwner;
	emit OwnershipTransferred(oldOwner, newOwner);
}
```

becames

```
event OwnershipTransferredTo(address indexed newOwner);
[...]
function setOwner(address newOwner) internal {
	emit OwnershipTransferredTo(owner = newOwner);
}
```


### Proof of concept with customized contracts

Using Remix IDE with optimization at 10k runs.

```
pragma solidity ^0.8.0;
contract Attempt{
	error NotOwner();
	event OwnershipTransferred(address  indexed previousOwner,  address  indexed newOwner);

	address  public owner;

	constructor(address _owner){
		owner = _owner;
	}

	function transferOwnership(address newOwner)  public onlyOwner {
		setOwner(newOwner);
	}

	function setOwner(address newOwner)  internal  {
		address oldOwner = owner;
		owner = newOwner;
		emit OwnershipTransferred(oldOwner, newOwner);
	}

	function requireOwner(address sender)  internal  view  {
		if  (owner != sender)  revert NotOwner();
	}

	  
	modifier onlyOwner()  {
		requireOwner(msg.sender);
		_;
	}
}

Execution cost of transferOwnership(): 4226 (if oldOwner==newOwner)
Execution cost of transferOwnership(): 7026 (if oldOwner!=newOwner)
Execution cost of transferOwnership(): 2445 (when it reverts)
```

```
pragma solidity ^0.8.0;

contract Attempt{
    error NotOwner();
    event OwnershipTransferredTo(address indexed newOwner);

    address public owner;

    constructor(address _owner){
        owner = _owner;
    }

    function transferOwnership(address newOwner) public onlyOwner {
        setOwner(newOwner);
    }
    
    function setOwner(address newOwner) internal {
        emit OwnershipTransferredTo(owner = newOwner);
    }

    function requireOwner(address sender) internal view {
        if (owner != sender) revert NotOwner();
    }

    modifier onlyOwner() {
        requireOwner(msg.sender);
        _;
    }
}

Execution cost of transferOwnership(): 3828 (if oldOwner==newOwner)
Execution cost of transferOwnership(): 6628 (if oldOwner!=newOwner)
Execution cost of transferOwnership(): 2445 (when it reverts)
```


## GAS-06 - Use right shift instead of multiplication by 2

### Founded in [MathUtil.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol)

* [MathUtil.sol#L18-L29](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L18-L29)

```
function _cubicRoot(uint256 _v) internal pure returns (uint256) {
	uint256 x = ONE_DEC18;
	uint256 xOld;
	bool cond;
	do {
		xOld = x;
		uint256 powX3 = _mulD18(_mulD18(x, x), x);
		x = _mulD18(x, _divD18( (powX3 + 2 * _v) , (2 * powX3 + _v)));
		cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
	} while ( cond );
	return x;
}
```

### Description
Using right shift saves gas with respect multiplication by 2.

```
function _cubicRoot(uint256 _v) internal pure returns (uint256) {
	uint256 x = ONE_DEC18;
	uint256 xOld;
	bool cond;
	do {
		xOld = x;
		uint256 powX3 = _mulD18(_mulD18(x, x), x);
		x = _mulD18(x, _divD18( (powX3 + 2 * _v) , (2 * powX3 + _v)));
		cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
	} while ( cond );
	return x;
}
```
becames
```
function _cubicRoot(uint256 _v) internal pure returns (uint256) {
	uint256 x = ONE_DEC18;
	uint256 xOld;
	bool cond;
	do {
		xOld = x;
		uint256 powX3 = _mulD18(_mulD18(x, x), x);
		x = _mulD18(x, _divD18( (powX3 + (_v << 1)) , ((powX3 << 1) + _v)));
		cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
	} while ( cond );
	return x;
}
```

### Proof of concept with customized contracts

Using Remix IDE with optimization at 10k runs.

```
pragma solidity ^0.8.0;

contract Attempt{
	uint256  internal  constant ONE_DEC18 =  10**18;
	uint256  internal  constant THRESH_DEC18 =  10000000000000000;//0.01

	function cubicRoot(uint256 _v)  public  returns  (uint256){
		return _cubicRoot(_v);
	}

	function _cubicRoot(uint256 _v)  internal  pure  returns  (uint256)  {
		uint256 x = ONE_DEC18;
		uint256 xOld;
		bool cond;
		do  {
			xOld = x;
			uint256 powX3 = _mulD18(_mulD18(x, x), x);
			x = _mulD18(x, _divD18(  (powX3 +  2  * _v)  ,  (2  * powX3 + _v)));
			cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
		}  while  ( cond );
		return x;
	}

	function _mulD18(uint256 _a,  uint256 _b)  internal  pure  returns(uint256)  {
		return _a * _b / ONE_DEC18;
	}
	  
	function _divD18(uint256 _a,  uint256 _b)  internal  pure  returns(uint256)  {
		return  (_a * ONE_DEC18)  / _b ;
	}

	function _power3(uint256 _x)  internal  pure  returns(uint256)  {
		return _mulD18(_mulD18(_x, _x), _x);
	}
}

Execution cost of cubicRoot(3): 9196 (it return 7812500000010531) 
Execution cost of cubicRoot(27*10**18): 5429 (it return 2999999999997076112) 
```

```
pragma solidity ^0.8.0;

contract Attempt{
    uint256 internal constant ONE_DEC18 = 10**18;
    uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01

    function cubicRoot(uint256 _v) public returns (uint256){
        return _cubicRoot(_v);
    }

    function _cubicRoot(uint256 _v) internal pure returns (uint256) {
            uint256 x = ONE_DEC18;
            uint256 xOld;
            bool cond;
            do {
                xOld = x;
                uint256 powX3 = _mulD18(_mulD18(x, x), x);
                x = _mulD18(x, _divD18( (powX3 + (_v << 1)) , ((powX3 << 1) + _v)));
                cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
            } while ( cond );
            return x;
        }

        function _mulD18(uint256 _a, uint256 _b) internal pure returns(uint256) {
            return _a * _b / ONE_DEC18;
        }

        function _divD18(uint256 _a, uint256 _b) internal pure returns(uint256) {
            return (_a * ONE_DEC18) / _b ;
        }

        function _power3(uint256 _x) internal pure returns(uint256) {
            return _mulD18(_mulD18(_x, _x), _x);
        }
}

Execution cost of cubicRoot(3): 8027 (it return 7812500000010531) 
Execution cost of cubicRoot(27*10**18):4761 (it return 2999999999997076112) 
```


## GAS-06 - Compute _power3 directly instead of call _mulD18 twice

### Founded in [MathUtil.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol)


* [MathUtil.sol#L39-L41](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L39-L41)

```
function _power3(uint256 _x) internal pure returns(uint256) {
	return _mulD18(_mulD18(_x, _x), _x);
}
```

### Description
Consider to directly compute _power3 instead of call _mulD18 twice

```
function _power3(uint256 _x) internal pure returns(uint256) {
	return _mulD18(_mulD18(_x, _x), _x);
}
```
becames
```
function _power3(uint256 _x) internal pure returns(uint256) {
	return _mulD18(_mulD18(_x, _x), _x);
	return _x ** 3 / ONE_DEC18 ** 2;
}
```

### Proof of concept with customized contracts

```
_mulD18(_a, _b) = _a * _b / ONE_DEC18
_mulD18(_x, _x) = _x * _x / ONE_DEC18 = _x**2 / ONE_DEC18
_mulD18(_mulD18(_x, _x), _x) = 
	(_x**2 / ONE_DEC18) * _x / ONE_DEC18 = _x**3 / ONE_DEC18**2

_power3(_x) = _x**3 / ONE_DEC18**2  or, better (multiplication cost less than power)

_power3(_x) = _x*_x*_x / ONE_DEC18*ONE_DEC18
```


Using Remix IDE with optimization at 10k runs.

```
pragma solidity ^0.8.0;

contract Attempt{
    uint256 internal constant ONE_DEC18 = 10**18;
    uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01

    function power3(uint256 _v) public returns (uint256){
        return _power3(_v);
    }

    function _cubicRoot(uint256 _v) internal pure returns (uint256) {
            uint256 x = ONE_DEC18;
            uint256 xOld;
            bool cond;
            do {
                xOld = x;
                uint256 powX3 = _mulD18(_mulD18(x, x), x);
                x = _mulD18(x, _divD18( (powX3 + 2 * _v) , (2 * powX3 + _v)));
                cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
            } while ( cond );
            return x;
        }

        function _mulD18(uint256 _a, uint256 _b) internal pure returns(uint256) {
            return _a * _b / ONE_DEC18;
        }

        function _divD18(uint256 _a, uint256 _b) internal pure returns(uint256) {
            return (_a * ONE_DEC18) / _b ;
        }

        function _power3(uint256 _x) internal pure returns(uint256) {
            return _mulD18(_mulD18(_x, _x), _x);
        }
}

Execution cost of power3(3 * 10**18): 681 (it return 27 * 10**18) 
```

```
pragma solidity ^0.8.0;

contract Attempt{
    uint256 internal constant ONE_DEC18 = 10**18;
    uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01

    function power3(uint256 _v) public returns (uint256){
        return _power3(_v);
    }

    function _cubicRoot(uint256 _v) internal pure returns (uint256) {
            uint256 x = ONE_DEC18;
            uint256 xOld;
            bool cond;
            do {
                xOld = x;
                uint256 powX3 = _mulD18(_mulD18(x, x), x);
                x = _mulD18(x, _divD18( (powX3 + 2 * _v) , (2 * powX3 + _v)));
                cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
            } while ( cond );
            return x;
        }

        function _mulD18(uint256 _a, uint256 _b) internal pure returns(uint256) {
            return _a * _b / ONE_DEC18;
        }

        function _divD18(uint256 _a, uint256 _b) internal pure returns(uint256) {
            return (_a * ONE_DEC18) / _b ;
        }

        function _power3(uint256 _x) internal pure returns(uint256) {
            return (_x*_x*_x)/(ONE_DEC18*ONE_DEC18);
        }
}

Execution cost of power3(3 * 10**18): 633 (it return 27 * 10**18) 
```


## GAS-07 - Consider to use different formula to compute _cubicRoot

### Founded in [MathUtil.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol)


* [MathUtil.sol#L18-L29](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L18-L29)

```

function _cubicRoot(uint256 _v) internal pure returns (uint256) {
	uint256 x = ONE_DEC18;
	uint256 xOld;
	bool cond;
	do {
		xOld = x;
		uint256 powX3 = _mulD18(_mulD18(x, x), x);
		x = _mulD18(x, _divD18( (powX3 + 2 * _v) , (2 * powX3 + _v)));
		cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
	} while ( cond );
	return x;
}
```

### Description

```
uint256 internal constant ONE_DEC18 = 10**18;
[...]
function _cubicRoot(uint256 _v) internal pure returns (uint256) {
	uint256 x = ONE_DEC18;
	uint256 xOld;
	bool cond;
	do {
		xOld = x;
		uint256 powX3 = _mulD18(_mulD18(x, x), x);
		x = _mulD18(x, _divD18( (powX3 + 2 * _v) , (2 * powX3 + _v)));
		cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
	} while ( cond );
	return x;
}
```
becames
```
uint256 internal constant ONE_DEC18 = 10**18;
uint256 internal constant ONE_DEC36 = 10**36;
[...]
function _cubicRoot(uint256 _v) internal pure returns (uint256) {
	uint256 x = ONE_DEC18;
	uint256 xOld;
	bool cond;
	do {
		xOld = x;
		x = x*(x*x*x + 2*ONE_DEC36*_v)/(2*x*x*x+ONE_DEC36*_v);
		cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
	} while ( cond );
	return x;
}
```

### Proof of concept

```
_mulD18(_a, _b) = _a * _b / ONE_DEC18
_divD18(_a, _b) = _a * ONE_DEC18 / _b
_power3(_x) = _x*_x*_x / ONE_DEC18*ONE_DEC18 = _x**3 / ONE_DEC36

(powX3 + 2 * _v) => (_x**3 / ONE_DEC36 + 2 * _v)
(2 * powX3 + _v) => (2*_x**3 / ONE_DEC36 + _v)

_divD18( (powX3 + 2 * _v) , (2 * powX3 + _v)) = 
	= ONE_DEC18*(_x**3 / ONE_DEC36 + 2 * _v)/(2*_x**3 / ONE_DEC36 + _v)

_mulD18(x, _divD18( (powX3 + 2 * _v) , (2 * powX3 + _v))) =
	= x*(ONE_DEC18*(_x**3 / ONE_DEC36 + 2 * _v)/(2*_x**3 / ONE_DEC36 + _v))/ONE_DEC18
	= x*(_x**3 / ONE_DEC36 + 2 * _v)/(2*_x**3 / ONE_DEC36 + _v)) =
	= ... =
	= x*(x**3 + 2*ONE_DEC36*_v)/(2*x**3+ONE_DEC36*_v)

	(multiplication is gas safer than power)

	= x*(x*x*x + 2*ONE_DEC36*_v)/(2*x*x*x+ONE_DEC36*_v)
```

Due to rounding, result will be bit different (I don't know if you would it).

Example:
```
_v = 27**10*18
Original result: 2999999999997076112 (execution cost 5429)
New result:      2999999999997076114 (execution cost 5077)
```

```
_v = 64**10*18
Original result: 3999999999999999999 (execution cost 6708)
New result:      3999999999999999999 (execution cost 6268)
```

```
_v = 1345**10*18
Original result: 11038433054931015055 (execution cost 7987)
New result:      11038433054931015063 (execution cost 7459)
```


It's important to check that _v is less than
```
Original: 10**58
New: about 27537**10*18
```
in order to avoid uint256 overflow


Execution cost computed by Remix IDE with optimization at 10k runs.

Original
```
pragma solidity ^0.8.0;

contract Attempt{
    uint256 internal constant ONE_DEC18 = 10**18;
    uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01

    function cubicRoot(uint256 _v) public returns (uint256){
        return _cubicRoot(_v);
    }

    function _cubicRoot(uint256 _v) internal pure returns (uint256) {
            uint256 x = ONE_DEC18;
            uint256 xOld;
            bool cond;
            do {
                xOld = x;
                uint256 powX3 = _mulD18(_mulD18(x, x), x);
                x = _mulD18(x, _divD18( (powX3 + 2 * _v) , (2 * powX3 + _v)));
                cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
            } while ( cond );
            return x;
        }

    function _mulD18(uint256 _a, uint256 _b) internal pure returns(uint256) {
        return _a * _b / ONE_DEC18;
    }

    function _divD18(uint256 _a, uint256 _b) internal pure returns(uint256) {
        return (_a * ONE_DEC18) / _b ;
    }

    function _power3(uint256 _x) internal pure returns(uint256) {
        return _mulD18(_mulD18(_x, _x), _x);
    }
}
```

New
```
pragma solidity ^0.8.0;

contract Attempt{
    uint256 internal constant ONE_DEC18 = 10**18;
    uint256 internal constant ONE_DEC36 = 10**36;
    uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01

    function cubicRoot(uint256 _v) public returns (uint256){
        return _cubicRoot(_v);
    }

    function _cubicRoot(uint256 _v) internal pure returns (uint256) {
            uint256 x = ONE_DEC18;
            uint256 xOld;
            bool cond;
            do {
                xOld = x;
                x = x*(x*x*x + 2*ONE_DEC36*_v)/(2*x*x*x+ONE_DEC36*_v);
                cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
            } while ( cond );
            return x;
        }

    function _mulD18(uint256 _a, uint256 _b) internal pure returns(uint256) {
        return _a * _b / ONE_DEC18;
    }

    function _divD18(uint256 _a, uint256 _b) internal pure returns(uint256) {
        return (_a * ONE_DEC18) / _b ;
    }

    function _power3(uint256 _x) internal pure returns(uint256) {
        return _mulD18(_mulD18(_x, _x), _x);
    }
}
```



## GAS-08 - Consider to use different if path inside onTokenTransfer()

### Founded in [StablecoinBridge.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol)


* [StablecoinBridge.sol#L75-L84](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L75-L84)

```
function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool){
	if (msg.sender == address(chf)){
		mintInternal(from, amount);
	} else if (msg.sender == address(zchf)){
		burnInternal(address(this), from, amount);
	} else {
		require(false, "unsupported token");
	}
	return true;
}
```

### Description
if/else if/else path could be different.

```
function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool){
	if (msg.sender == address(chf)){
		mintInternal(from, amount);
	} else if (msg.sender == address(zchf)){
		burnInternal(address(this), from, amount);
	} else {
		require(false, "unsupported token");
	}
	return true;
}
```

becames

```
function onTokenTransfer(address from, uint256 amount) external returns (bool){
	require(msg.sender == address(chf) || msg.sender == address(zchf), "unsupported token");
	if (msg.sender == address(chf)){
		mintInternal(from, amount);
	} else{
		burnInternal(address(this), from, amount);
	}
	return true;
}
```

### Proof of concept

```
if (condition1){action1;}
else if (condition2){action2;}
else {action3;}
```

is same of

```
if(!condition1 && !condition2){
	action3;
}
else{
	//condition1 = true || condition2 = true
	if (condition1){action1;}
	else{
		//condition1 = false => condition2 = true
		action2;
	}
}

```

is same of

```
if(!(condition1 || condition2)){
	action3;
}
else{
	if (condition1){action1;}
	else{action2;}
}

```

is same of

```
require(condition1 || condition2, action3);
if (condition1){action1;}
else{action2;}

```

### Test  with customized contracts

Used Remix IDE with optimization at 10k runs.

Original
```
pragma solidity ^0.8.0;

contract Attempt{

    function mintInternal(address target, uint256 amount) internal{

    }

    function burnInternal(address zchfHolder, address target, uint256 amount) internal{

    }

    function onTokenTransfer(address from, uint256 amount) external returns (bool){
        if (msg.sender == address(chf)){
            mintInternal(from, amount);
        } else if (msg.sender == address(zchf)){
            burnInternal(address(this), from, amount);
        } else {
            require(false, "unsupported token");
        }
        return true;
    }
}
```

New
```
pragma solidity ^0.8.0;

contract Attempt{

    address chf = 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4;
    address zchf = 0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2;

    function mintInternal(address target, uint256 amount) internal{

    }

    function burnInternal(address zchfHolder, address target, uint256 amount) internal{

    }

    function onTokenTransfer(address from, uint256 amount) external returns (bool){
        require(msg.sender == address(chf) || msg.sender == address(zchf), "unsupported token");
        if (msg.sender == address(chf)){
            mintInternal(from, amount);
        } else{
            burnInternal(address(this), from, amount);
        }
        return true;
    }
}
```

Result:

msg.sender == address(chf) 
```
Original, execution cost: 2463
New, execution cost: 2468
```

msg.sender == address(zchf) 
```
Original, execution cost: 4594
New, execution cost: 4584
```

msg.sender != address(chf) && msg.sender != address(zchf)
```
Original, execution cost: 4605
New, execution cost: 4607
```

saving an average of 3 gas.




## GAS-09 - Avoid to check transfer() return value

### Founded in [ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol)


* [ERC20.sol#L162-L168](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L162-L168)

```
// ERC-677 functionality, can be useful for swapping and wrapping tokens
function transferAndCall(address recipient, uint256 amount, bytes calldata data) external override returns (bool) {
	bool success = transfer(recipient, amount);
	if (success){
		success = IERC677Receiver(recipient).onTokenTransfer(msg.sender, amount, data);
	}
	return success;
}
```

caused by

* [ERC20.sol#L77-L88](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L77-L88)

```
/**
* @dev See `IERC20.transfer`.
*
* Requirements:
*
* - `recipient` cannot be the zero address.
* - the caller must have a balance of at least `amount`.
*/
function transfer(address recipient, uint256 amount) public virtual override returns (bool) {
	_transfer(msg.sender, recipient, amount);
	return true;
}

```

### Description
transfer(address recipient, uint256 amount) returns always true, or transaction will revert (due to checks inside _transfer).
So, it's useless to check transfer return value.

```
// ERC-677 functionality, can be useful for swapping and wrapping tokens
function transferAndCall(address recipient, uint256 amount, bytes calldata data) external override returns (bool) {
	bool success = transfer(recipient, amount);
	if (success){
		success = IERC677Receiver(recipient).onTokenTransfer(msg.sender, amount, data);
	}
	return success;
}
```

becames

```
// ERC-677 functionality, can be useful for swapping and wrapping tokens
function transferAndCall(address recipient, uint256 amount, bytes calldata data) external override returns (bool) {
	transfer(recipient, amount);
	return IERC677Receiver(recipient).onTokenTransfer(msg.sender, amount, data);
}
```


### Proof of concept with customized contracts

Used Remix IDE with optimization at 10k runs.

Original
```
pragma solidity ^0.8.0;

contract Attempt{

    function transfer(address recipient, uint256 amount) public returns(bool){
        require(recipient != address(0));
        return true;
    }

    function IERC677Receiver_onTokenTransfer(address recipient, uint256 amount) public returns(bool){
        require(recipient != address(0));

        return true;
    }

    function transferAndCall(address recipient, uint256 amount) external returns (bool) {
        bool success = transfer(recipient, amount);
        if (success){
            success = IERC677Receiver_onTokenTransfer(recipient, amount);
        }
        return success;
    }
}

Execution cost of transferAndCall(): 495
```

New
```
pragma solidity ^0.8.0;

contract Attempt{

    function transfer(address recipient, uint256 amount) public returns(bool){
        require(recipient != address(0));
        return true;
    }

    function IERC677Receiver_onTokenTransfer(address recipient, uint256 amount) public returns(bool){
        require(recipient != address(0));
        return true;
    }

    function transferAndCall(address recipient, uint256 amount) external returns (bool) {
        transfer(recipient, amount);
        return IERC677Receiver_onTokenTransfer(recipient, amount);
    }
}

Execution cost of transferAndCall(): 464
```



## GAS-10 - Consider to rewrite Frankencoin.sol:allowanceInternal()

### Founded in [Frankencoin.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol)


* [Frankencoin.sol#L102-L111](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L102-L111)

```
function allowanceInternal(address owner, address spender) internal view override returns (uint256) {
	uint256 explicit = super.allowanceInternal(owner, spender);
	if (explicit > 0){
		return explicit; // don't waste gas checking minter
	} else if (isMinter(spender) || isMinter(isPosition(spender))){
		return INFINITY;
	} else {
		return 0;
	}
}
```

### Description
You could rewrite if/elseif/else condition remembering that explicit is uint256 (unsigned), and so it is >=0.

```
function allowanceInternal(address owner, address spender) internal view override returns (uint256) {
	uint256 explicit = super.allowanceInternal(owner, spender);
	if (explicit > 0){
		return explicit; // don't waste gas checking minter
	} else if (isMinter(spender) || isMinter(isPosition(spender))){
		return INFINITY;
	} else {
		return 0;
	}
}
```

becames

```
function allowanceInternal(address owner, address spender) internal view override returns (uint256) {
	uint256 explicit = super.allowanceInternal(owner, spender);
	if (explicit==0){
		if (isMinter(spender)) return INFINITY;
		if (isMinter(isPosition(spender))) return INFINITY;
	}
	return explicit;
}
```


### Proof of concept

We have 4 cases:

#### CASE 1
```
explicit == 0 && isMinter(spender) || isMinter(isPosition(spender))

In this case Original function return INFINITY
In this case New function return INFINITY
```

#### CASE 2
```
explicit == 0 && !(isMinter(spender) || isMinter(isPosition(spender)))

In this case Original function return 0
In this case New function return explicit (which is 0)
```

#### CASE 3
```
explicit > 0 && isMinter(spender) || isMinter(isPosition(spender))

In this case Original function return explicit
In this case New function return explicit
```

#### CASE 4
```
explicit > 0 && !(isMinter(spender) || isMinter(isPosition(spender)))

In this case Original function return explicit
In this case New function return explicit
```

### Test with customized contracts

Used Remix IDE with optimization at 10k runs.

Original
```
pragma solidity ^0.8.0;

contract Attempt{

    uint256 public INFINITY = 100;

    function tryGas(uint256 explicit, bool isMinter_spender, bool isMinter_isPosition_spender) public returns(uint256){
        if (explicit > 0){
            return explicit; // don't waste gas checking minter
        } else if (isMinter_spender || isMinter_isPosition_spender){
            return INFINITY;
        }
    }
}
```

New
```
pragma solidity ^0.8.0;

contract Attempt{

    uint256 public INFINITY = 100;

    function tryGas(uint256 explicit, bool isMinter_spender, bool isMinter_isPosition_spender) public returns(uint256){
        if (explicit == 0){
            if(isMinter_spender) return INFINITY;
            if(isMinter_isPosition_spender) return INFINITY;
        }
        return explicit;
    }
}
```

#### CASE 1
```
Original execution cost: 2610
New execution cost: 2606
```

#### CASE 2
```
Original execution cost: 510
New execution cost: 516
```

#### CASE 3
```
Original execution cost: 484
New execution cost: 477
```

#### CASE 4
```
Original execution cost: 484
New execution cost: 477
```