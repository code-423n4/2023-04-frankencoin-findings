# LOW RISK or NOT CRITICAL

## N-01 - Errors declared but not used

### Founded in [Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol)

* [Position.sol#L104](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L104)

```
error NotQualified();
```

### Description
NotQualified() is not used. 


## N-02 - Consider to rename _power3(uint256 _x) internal function

### Founded in [MathUtil.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol)

* [MathUtil.sol#L39-L41](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L39-L41)

```
function _power3(uint256 _x) internal pure returns(uint256) {
	return _mulD18(_mulD18(_x, _x), _x);
}
```

### Description

Consider to rename _power3 to _power3D18, because it doesn't return power3 of _x, it returns 
```
((_x)**3)/(ONE_DEC18**2)
```


## N-03 - Use defined function _power3

### Founded in [MathUtil.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol)

* [MathUtil.sol#L24](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MathUtil.sol#L24)

```
uint256 powX3 =  _mulD18(_mulD18(x, x), x);
```

### Description
Use defined function _power3:
```
uint256 powX3 =  _mulD18(_mulD18(x, x), x);
```

becames

```
uint256 powX3 =  _power3(x);
```


## N-04 - Avoid to compute recoveredAddress if owner==address(0)

### Founded in [ERC20PermitLight.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol)

* [ERC20PermitLight.sol#L21-L59](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L21-L59)

```
function permit(
	address owner,
	address spender,
	uint256 value,
	uint256 deadline,
	uint8 v,
	bytes32 r,
	bytes32 s
) public {
	require(deadline >= block.timestamp, "PERMIT_DEADLINE_EXPIRED");

	unchecked { // unchecked to save a little gas with the nonce increment...
		address recoveredAddress = ecrecover(
			keccak256(
				abi.encodePacked(
					"\x19\x01",
					DOMAIN_SEPARATOR(),
					keccak256(
						abi.encode(
							// keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),
							bytes32(0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9),
							owner,
							spender,
							value,
							nonces[owner]++,
							deadline
						)
					)
				)
			),
			v,
			r,
			s
		);

		require(recoveredAddress != address(0) && recoveredAddress == owner, "INVALID_SIGNER");
		_approve(recoveredAddress, spender, value);
	}
}
```

### Description

Use different require check in order to avoid recoveredAddress computation when onwer==address(0)

```
recoveredAddress != address(0) && recoveredAddress == owner
```

is same of

```
owner != address(0) && recoveredAddress == owner
```

because it's an AND of two sentences, so if it's true, recoveredAddress == owner and so we can write "owner" instead of "recoveredAddress" in first sentence

As discussed in [CodingNameKiki/AutomatedFindings.md - [GAS‑19] Splitting require() statements that use && saves gas](https://gist.github.com/CodingNameKiki/36f3bfb214907d68fdf3a43cb0cb8ae3#GAS-19),
it's gas safe splitting require statement.

So we obtain:

```
require(owner != address(0));
require(recoveredAddress == owner);
```

I don't know if there is real possibility that owner == address(0).

If it can happen, it's better if you check owner address before than compute recoveredAddress.


function permit() becames:

```
function permit(
	address owner,
	address spender,
	uint256 value,
	uint256 deadline,
	uint8 v,
	bytes32 r,
	bytes32 s
) public {
	require(deadline >= block.timestamp, "PERMIT_DEADLINE_EXPIRED");
	require(owner != address(0), "INVALID_SIGNER");

	unchecked { // unchecked to save a little gas with the nonce increment...
		address recoveredAddress = ecrecover(
			keccak256(
				abi.encodePacked(
					"\x19\x01",
					DOMAIN_SEPARATOR(),
					keccak256(
						abi.encode(
							// keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),
							bytes32(0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9),
							owner,
							spender,
							value,
							nonces[owner]++,
							deadline
						)
					)
				)
			),
			v,
			r,
			s
		);

		require(recoveredAddress == owner, "INVALID_SIGNER");
		_approve(recoveredAddress, spender, value);
	}
}
```


## N-04 - function approve() returns always true

### Founded in [ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol)

* [ERC20.sol#L108-L111](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L108-L111)

```
function approve(address spender, uint256 value) external override returns (bool) {
	_approve(msg.sender, spender, value);
	return true;
}
```

### Description
Function approve(address spender, uint256 value) returns only true.

**approve(address spender, uint256 value)** calls only **_approve(address owner, address spender, uint256 value)**

* [ERC20.sol#L208-L224](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L208-L224)
```
/**
* @dev Sets `amount` as the allowance of `spender` over the `owner`s tokens.
*
* This is internal function is equivalent to `approve`, and can be used to
* e.g. set automatic allowances for certain subsystems, etc.
*
* Emits an `Approval` event.
*
* Requirements:
*
* - `owner` cannot be the zero address.
* - `spender` cannot be the zero address.
*/
function _approve(address owner, address spender, uint256 value) internal {
	_allowances[owner][spender] = value;
	emit Approval(owner, spender, value);
}
```

There are requirements, but there aren't checks of those requirements inside _approve().
When you call _approve() in other contracts (like in [ERC20PermitLight.sol#L57](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L57)), you check those requirements outside, but it could be better if you put require() inside _approve().

Anyway, as you wrote, approve() returns always true and you could think to remove return value.

```
function approve(address spender, uint256 value) external override returns (bool) {
	_approve(msg.sender, spender, value);
	return true;
}
```

becames

```
function approve(address spender, uint256 value) external{
	_approve(msg.sender, spender, value);
	return true;
}
```

or, better, in order to follow ERC20 standard,

```
function approve(address spender, uint256 value) external override returns (bool) {
	if(msg.sender == address(0) || spender == address(0)){
		return false;
	}

	_approve(msg.sender, spender, value);
	return true;
}
```