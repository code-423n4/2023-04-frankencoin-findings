## [G-01] Use assembly to check for address(0)

Checking zero address can be improved by replacing the require statement with Assembly.Solidity has a lot of guardrails that can be removed (with care) for optimization purposes, especially for simple functionality like checking if an address is zero.

    File: contracts/ERC20.sol

    152: require(recipient != address(0));

    180: require(recipient != address(0));

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol

    File: contracts/ERC20PermitLight.sol

    56: require(recoveredAddress != address(0) && recoveredAddress == owner, "INVALID_SIGNER");

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol

## [G-02] Use nested if and, avoid multiple check combinations

Using nesting is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

    File: contracts/Position.sol

    294: if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

    File: contracts/Frankencoin.sol

    84: if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();

    85: if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();

    267: if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

## [G-03] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

    File: contracts/Position.sol

    141: if (newMinted < minted){
    142:     zchf.burnFrom(msg.sender, minted - newMinted, reserveContribution);
    143:     minted = newMinted;
    144: }
    145: if (newCollateral < colbal){
    146:     withdrawCollateral(msg.sender, colbal - newCollateral);
    147: }
    148: // Must be called after collateral withdrawal
    149: if (newMinted > minted){
    150:     mint(msg.sender, newMinted - minted);

    187: return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));

    241: if (amount > minted) revert RepaidTooMuch(amount - minted);

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

    File: contracts/Frankencoin.sol

    153: if (block.timestamp > minters[_minter]) revert TooLate();
    154: reserve.checkQualified(msg.sender, _helpers);
    155: delete minters[_minter];

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

## [G-04] The result of function calls should be cached rather than re-calling the function

The instances below point to the second+ call of the function within a single function. Every external call made to a contract incurs at least 100 gas of overhead.

    File: contracts/Equity.sol

    145: uint256 lostVotes = from == address(0x0) ? 0 : (anchorTime() - voteAnchor[from]) * amount;
    146: totalVotesAtAnchor = uint192(totalVotes() - roundingLoss - lostVotes);
    147: totalVotesAnchorTime = anchorTime();

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

    File: contracts/Frankencoin.sol

    84: if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
    85: if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();

    207: if (currentReserve < minterReserve()){
    208:    // not enough reserves, owner has to take a loss
    209:    return theoreticalReserve * currentReserve / minterReserve();

    281: uint256 reserveLeft = balanceOf(address(reserve));
    282: if (reserveLeft >= _amount){
    283:    _transfer(address(reserve), msg.sender, _amount);
    284: } else {
    285:    _transfer(address(reserve), msg.sender, reserveLeft);
    286:    _mint(msg.sender, _amount - reserveLeft);
    287: }

    294: return minters[_minter] != 0 && block.timestamp >= minters[_minter];

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

    File: contracts/ERC20.sol

    155: if (_balances[sender] < amount) revert ERC20InsufficientBalance(sender, _balances[sender], amount);
    156: _balances[sender] -= amount;

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol

## [G-05] Instead of counting down in for statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

     File: contracts/Equity.sol

     192: for (uint i=0; i<helpers.length; i++){

     312: for (uint256 i = 0; i<addressesToWipe.length; i++){

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

## [G-06] Use != 0 instead of > 0

Using > 0 costs more gas than != 0 when used on a uint.

     File: contracts/Position.sol

     381: if (challengedAmount > 0) revert Challenged();

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

     File: contracts/MintingHub.sol

     203: if (challenge.bid > 0) {

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

     File: contracts/Equity.sol

     114: if (amount > 0){

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

     File: contracts/Frankencoin.sol

     84: if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();

     85: if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();

     104: if (explicit > 0){

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol

## [G-07] Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend.

     File: contracts/MintingHub.sol

     282: uint256 amount = pendingReturns[collateral][msg.sender];

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

     File: contracts/Equity.sol

     294: uint256 newTotalShares = totalShares - shares;

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

## [G-08] Instead of caching function call result, directly call the function

If any function call is required only once inside the function then instead of caching it's result in a variable directly call the function.

     File: contracts/Position.sol

     233: uint256 actuallyBurned = IFrankencoin(zchf).burnWithReserve(burnable, reserveContribution);

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol

     File: contracts/MintingHub.sol

     126: uint256 limit = existing.reduceLimitForClone(_initialMint);

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol

     File: contracts/Equity.sol

     210: uint256 _votes = votes(sender, helpers);

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol

     File: contracts/PositionFactory.sol

     31: Position existing = Position(_existing);

     32: Position clone = Position(createClone(existing.original()));

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/PositionFactory.sol

