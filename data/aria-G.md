[GAS-01] - Suggest the first Minter of Frankencoin in the constructor and save some gas in the suggestMinter() Function

before:
```java
function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {
   if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
   if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
   if (minters[_minter] != 0 && minters[_minter] <= block.timestamp + _applicationPeriod) revert AlreadyRegistered();
   _transfer(msg.sender, address(reserve), _applicationFee);
   minters[_minter] = block.timestamp + _applicationPeriod;
   emit MinterApplied(_minter, _applicationPeriod, _applicationFee, _message);
}
```

after:
```java
function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {
   if (_applicationPeriod < MIN_APPLICATION_PERIOD) revert PeriodTooShort();
   if (_applicationFee < MIN_FEE) revert FeeTooLow();
   if (minters[_minter] != 0) revert AlreadyRegistered();
   _transfer(msg.sender, address(reserve), _applicationFee);
   minters[_minter] = block.timestamp + _applicationPeriod;
   emit MinterApplied(_minter, _applicationPeriod, _applicationFee, _message);
}
```
in constructor - 

```java
constructor(uint256 _minApplicationPeriod, address _initialMinter) ERC20(18){
   MIN_APPLICATION_PERIOD = _minApplicationPeriod;
   reserve = new Equity(this);

   minters[_initialMinter] = block.timestamp;
   emit MinterApplied(_initialMinter, 0, 0, "Initial minter"); 
}
```

scope - 
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L83-L90

[GAS - 02] - 
function isClosed is wasting gas on deployment - both fields (collateralBalance which is a wrapper of IERC20.balanceof(address(this))) and minimumCollateral are public.

If this function is only used on front end there is no reason to waste gas on it

before:

```
function  isClosed() public  view  returns (bool) {
	return  collateralBalance() < minimumCollateral;
}
```
after:

```java
//nothing///
```
scope:
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L360-L362

[GAS - 03] - 
For medium to large amount of helpers, it is better to test if someone already voted by using a storage map.
reducing the time complexity to O(n) from O(n^2)

before:
```java
function votes(address sender, address[] calldata helpers) public view returns (uint256) {
        uint256 _votes = votes(sender);
        for (uint i=0; i<helpers.length; i++){
            address current = helpers[i];
            require(current != sender);
            require(canVoteFor(sender, current));
            for (uint j=i+1; j<helpers.length; j++){
                require(current != helpers[j]); // ensure helper unique
            }
            _votes += votes(current);
        }
        return _votes;
}
```

```java
mapping(address => bool) counted;
function votes(address sender, address[] calldata helpers) public view returns (uint256) {
        uint256 _votes = votes(sender);
        counted[sender] = true;
        
        for (uint i=0; i<helpers.length; i++){
            address current = helpers[i];
            
            require(canVoteFor(sender, current));
            require(!counted[current]);
            
            counted[current] = true;
            _votes += votes(current);
        }

		for (uint i = 0; i<helpers.length; i++){
			counted[helpers[i] = false;
		}
		counted[sender] = false;
        return _votes;
}
```

scope:
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L190-L202