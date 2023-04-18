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

## [NC-01] Unclear variable naming

Confusing variable naming in the constructor.
Consider changing `other` to `sourceStablecoin` as the comment suggests.
Much clearer for devs/auditer to understand what the variable is.

```
// StablecoinBridge.sol

contract StablecoinBridge {

    IERC20 public immutable chf; // the source stablecoin

    constructor(address other, address zchfAddress, uint256 limit_){
        chf = IERC20(other);
        zchf = IFrankencoin(zchfAddress);
        horizon = block.timestamp + 52 weeks;
        limit = limit_;
    }
// ...
```

## [NC-02] Overly complicated code

The following code seems overly complicated for what it is trying to acheive. It is not called elsewhere in the supplied contracts and therefore could be simplified.

```
function requireOwner(address sender) internal view {
    if (owner != sender) revert NotOwner();
}

modifier onlyOwner() {
    requireOwner(msg.sender);
    _;
}
```

```
// leaner version of the above
modifier onlyOwnerAlternative() {
    if (owner != msg.sender) revert NotOwner();
    _;
}
```

Sponsor may have legitimate reasons for this code, but it is not clear from the supplied contracts.
