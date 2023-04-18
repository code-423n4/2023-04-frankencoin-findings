## [L-01] Leaving contract vulnerable to a simple mistake

In the function and the comment, we leave the power in the users hands to ensure a mistake is not made. In an ideal world this would be fine - creators would ensure the only call it once but why not make it challenging to do so.

```
contract InitialiseClone {

    /**
     * Method to initialize a freshly created clone. It is the responsibility of the creator to make sure this is only
     * called once and to call reduceLimitForClone on the original position before initializing the clone.
     */

    function initializeClone(address owner, uint256 _price, uint256 _limit, uint256 _coll, uint256 _mint)
    external onlyHub {
        if(_coll < minimumCollateral) revert InsufficientCollateral();
        setOwner(owner);
        price = _mint * ONE_DEC18 / _coll;
        if (price > _price) revert InsufficientCollateral();
        limit = _limit;
        mintInternal(owner, _mint, _coll);
        emit PositionOpened(owner, original, address(zchf), address(collateral), _price);
    }
```

Recommendation: Add in some method to check if the clone has been initialised or not. This could be something like a boolean or a mapping.
```
//e.g.

mapping(address owner => mapping(Position position  => bool initialised)) public initialisedClones;
```

Users that are new to the protocol could easily make a mistake, or malicious actors may con another user into making the mistake. 

## [L-02] What happens if we set a random address to the owner?

Should the owner variable be open to assignable to anyone, if we are Alice and purposefully or accidentally set the owner to Bob, what happens?

```
function initializeClone(address owner, uint256 _price, uint256 _limit, uint256 _coll, uint256 _mint)
    external onlyHub {
        if(_coll < minimumCollateral) revert InsufficientCollateral();

        setOwner(owner);

        price = _mint * ONE_DEC18 / _coll;
        if (price > _price) revert InsufficientCollateral();
        limit = _limit;
        mintInternal(owner, _mint, _coll);
        emit PositionOpened(owner, original, address(zchf), address(collateral), _price);
    }
```

Is there a good reason as to why it should not simply be msg.sender/hub?

