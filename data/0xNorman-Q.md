## LOW-1: Potential front-running vulnerability in _approve()
approve() may be front-run. 
In the ERC20.sol, approve() may be front-run. Consider using```increaseAllowance()``` as the best practice.

Proof Of Concept:
```
function _approve(address owner, address spender, uint256 value) internal {
        _allowances[owner][spender] = value;
        emit Approval(owner, spender, value);
    }
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L221-L224
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L108-L111

## LOW-2: Redeem function missing zero address check
In the Frankencoin.sol, the function ```redeem()``` can be used as burn function due to lack of zero address check. 
Proof of Concept:
```
function redeem(address target, uint256 shares) public returns (uint256) {
        require(canRedeem(msg.sender));
        uint256 proceeds = calculateProceeds(shares);
        _burn(msg.sender, shares);
        zchf.transfer(target, proceeds);
        emit Trade(msg.sender, -int(shares), proceeds, price());
        return proceeds;
    }
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L275-L282

## LOW-4 Arbitrary burning and sending of source coin
In the StablecoinBridge.sol, the function ```burn()``` will allow anyone to burn any amount of Frankencoin and send the same number of source coin to the arbitrary address.  And due to lack of the zero address check, this can cause the loss of the source coin. If this is not the intended behavior, consider adjust the code to match the comment.
```
/**
     * Burn the indicated amount of Frankencoin and send the same number of source coin to the caller.
     * No allowance required.
     */
 function burn(address target, uint256 amount) external {
        burnInternal(msg.sender, target, amount);
    }
function burnInternal(address zchfHolder, address target, uint256 amount) internal {
        zchf.burn(zchfHolder, amount);
        chf.transfer(target, amount);
    }
```
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L63-L70
## LOW-5 Miss the zero address check in the StablecoinBridge.sol for both ```burn()``` and ```burnFrom()```. Consider add this check.
In ERC20.sol, both ```approve(),_approve()``` miss the zero address check for spender. Consider add this check.
```
function approve(address spender, uint256 value) external override returns (bool) {
        _approve(msg.sender, spender, value);
        return true;
    }
function _approve(address owner, address spender, uint256 value) internal {
        _allowances[owner][spender] = value;
        emit Approval(owner, spender, value);
    }
```



## NC-1
1. Variable ```IFrankencoin.burnWithReserve(uint256,uint32).amountExcludingReserve``` (IFrankencoin.sol#36) is too similar to ```IFrankencoin.burn(uint256,uint32).amountIncludingReserve``` (IFrankencoin.sol#32). 
Consider renaming it to avoid confusion.
## NC-2 Missing events for arithmetic operations in Frankencoin.sol
```
Frankencoin.mint(address,uint256,uint32,uint32) (Frankencoin.sol#165-170) should emit an event for: 
        - minterReserveE6 += _amount * _reservePPM (Frankencoin.sol#169) 
Frankencoin.burn(uint256,uint32) (Frankencoin.sol#194-197) should emit an event for: 
        - minterReserveE6 -= amount * reservePPM (Frankencoin.sol#196) 
Frankencoin.burnFrom(address,uint256,uint32) (Frankencoin.sol#223-229) should emit an event for: 
        - minterReserveE6 -= targetTotalBurnAmount * _reservePPM (Frankencoin.sol#227) 
Frankencoin.burnWithReserve(uint256,uint32) (Frankencoin.sol#251-257) should emit an event for: 
        - minterReserveE6 -= freedAmount * _reservePPM (Frankencoin.sol#253) 
```
## NC-3 Using too many literals
Proof of Concept:
```
Equity.slitherConstructorConstantVariables() (Equity.sol#20-319) uses literals with too many digits:
        - THRESH_DEC18 = 10000000000000000 (MathUtil.sol#11)
Frankencoin.minterReserve() (Frankencoin.sol#117-119) uses literals with too many digits:
        - minterReserveE6 / 1000000 (Frankencoin.sol#118)
Frankencoin.calculateAssignedReserve(uint256,uint32) (Frankencoin.sol#205-214) uses literals with too many digits:
        - theoreticalReserve = _reservePPM * mintedAmount / 1000000 (Frankencoin.sol#206)
Frankencoin.calculateFreedAmount(uint256,uint32) (Frankencoin.sol#236-241) uses literals with too many digits:
        - 1000000 * amountExcludingReserve / (1000000 - adjustedReservePPM) (Frankencoin.sol#240)
```
Use:
 Ether suffix,
    Time suffix, or
    The scientific notation
