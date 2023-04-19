## [LC-1] VARIABLE SHADOWING
The `owner` argument of the `initializeClone` function in Position.sol shadows the owner state variable in the Ownable contract that Position.sol inherited.

File: https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L76

```
function initializeClone(address owner, uint256 _price, uint256 _limit, uint256 _coll, uint256 _mint)
        external
        onlyHub
    {
        if (_coll < minimumCollateral) revert InsufficientCollateral();
        setOwner(owner); //@audit state variable shadowing

        price = _mint * ONE_DEC18 / _coll;
        if (price > _price) revert InsufficientCollateral();
        limit = _limit;
        mintInternal(owner, _mint, _coll);

        emit PositionOpened(owner, original, address(zchf), address(collateral), _price);
    }

```

#### Recommended Mitigation Steps.
Consider renaming the `owner` argument on the `initializeClone` function in the Position.sol file to `_owner`.

## [NC-1] USE revert() INSTEAD OF require(false).
The code in the `else` block of the  `onTokenTransfer` function in the `StablecoinBridge.sol` would be better if the `require(false)` is replaced with revert().

File: https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L81

```
function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool){
        if (msg.sender == address(chf)){
            mintInternal(from, amount);
        } else if (msg.sender == address(zchf)){
            burnInternal(address(this), from, amount);
        } else {
81:            require(false, "unsupported token");
        }
        return true;
    }
```

## [NC-2] ADD A FUNCTIONALITY TO ADJUST OPENING FEE
The `OPENING_FEE` is fixed at 1000 * 10 ** 18. Add a function to adjust opening fee with a limit check.
There may be need to adjust the `OPENING_FEE in the future due to changing value of assets with time.

File: https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L20

```
uint256 public constant OPENING_FEE = 1000 * 10**18;
```