# Low Severity Issues

## Position can be created with 0 initial period

When creating a position an initial period is given to allow other system participants to veto the new position. But a position with 0 initial period can be created to avoid being vetod.

`_initPeriodSeconds` is not validated, so it can be 0

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L88

```
    function openPosition(
        address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
        uint256 _mintingMaximum, uint256 _initPeriodSeconds, uint256 _expirationSeconds, uint256 _challengeSeconds,
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address) {
        IPosition pos = IPosition(
            POSITION_FACTORY.createNewPosition(
                msg.sender,
                address(zchf),
                _collateralAddress,
                _minCollateral,
                _mintingMaximum,
                _initPeriodSeconds,
                _expirationSeconds,
                _challengeSeconds,
                _mintingFeePPM,
                _liqPrice,
                _reservePPM
            )
        );

```

Position#deny will revert when pool shareholders try to expire the newly created position

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L109

```
    function deny(address[] calldata helpers, string calldata message) public {
        if (block.timestamp >= start) revert TooLate();
```
## Owner can renounce ownership

Owner can renounce ownership which will cause revert in functions that contains onlyOwner modifier 

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L31

```
    function transferOwnership(address newOwner) public onlyOwner {
        setOwner(newOwner);
    }
```

Add conditon to prevent ownership being transferred to address(0)

```
    function transferOwnership(address newOwner) public onlyOwner {
        if(newOwner == address(0)) InvalidAddress();
        setOwner(newOwner);
    }
```

## Ownership transfer can be a 2 step procedure

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L39

```
    function transferOwnership(address newOwner) public onlyOwner {
        setOwner(newOwner);
    }

    function setOwner(address newOwner) internal {
        address oldOwner = owner;
        owner = newOwner;
        emit OwnershipTransferred(oldOwner, newOwner);
    }
```

## Missing zero division check

Missing zero value check on variable `_b` will cause function to revert silently  

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MathUtil.sol#L36

```
    function _divD18(uint256 _a, uint256 _b) internal pure returns(uint256) {
        return (_a * ONE_DEC18) / _b ;
    }
```

Add checks to avoid division by 0

```
    if(_b == 0) revert DivByZero();
```

# Non-critical Issues

## Floating pragma

Contract should be deployed with the same version they were tested with thoroughy. Locking pragma ensures that contract do not accidentally get deployed with an outdated version

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L2

```
    pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L2

```
    pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L4

```
    pragma solidity >=0.8.0 <0.9.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L9

```
    pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MathUtil.sol#L3

```
    pragma solidity >=0.8.0 <0.9.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L12

```
    pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L5

```
    pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L2

```
    pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L2

```
    pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L2

```
    pragma solidity ^0.8.0;
```

## Missing revert messages

Require statements are missing revert messages, if the require condition fails the function will fail silently without error message

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol

```
L180        require(recipient != address(0));

L152        require(recipient != address(0));
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L158

```
        require(challenge.challenger != address(0x0));
        ...
        require(challenge.size >= min);
        require(copy.size >= min);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L254

```
        require(challenge.challenger != address(0x0));
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L310

```
        require(zchf.equity() < MINIMUM_EQUITY);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L276


```
        require(canRedeem(msg.sender)); 
```

## Incomplete natspec comments

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L162

```
    // ERC-677 functionality, can be useful for swapping and wrapping tokens
    function transferAndCall(address recipient, uint256 amount, bytes calldata data) external override returns (bool)
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L59

```
   /**
    * Initiates the Frankencoin with the provided minimum application period for new plugins
    * in seconds, for example 10 days, i.e. 3600*24*10 = 864000
    */
   constructor(uint256 _minApplicationPeriod) ERC20(18)
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L117

```
   /**
    * The reserve provided by the owners of collateralized positions.
    * The minter reserve can be used to cover losses after all else failed and the equity holders have already been wiped out.
    */
   function minterReserve() public view returns (uint256) 
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L125

```
   /**
    * Registers a collateralized debt position, thereby giving it the ability to mint Frankencoins.
    * It is assumed that the responsible minter that registers the position ensures that the position can be trusted.
    */
   function registerPosition(address _position) override external
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L138

```
   /**
    * The amount of equity of the Frankencoin system in ZCHF, owned by the holders of Frankencoin Pool Shares.
    * Note that the equity contract technically holds both the minter reserve as well as the equity, so the minter
    * reserve must be subtracted. All fees and other kind of income is added to the Equity contract and essentially
    * constitutes profits attributable to the pool share holders.
    */
   function equity() public view returns (uint256)
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L152

```
   /**
    * Qualified pool share holders can deny minters during the application period.
    * Calling this function is relatively cheap thanks to the deletion of a storage slot.
    */
   function denyMinter(address _minter, address[] calldata _helpers, string calldata _message) override external 
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L165

```
   /**
    * Mints the provided amount of ZCHF to the target address, automatically forwarding
    * the minting fee and the reserve to the right place.
    */
   function mint(address _target, uint256 _amount, uint32 _reservePPM, uint32 _feesPPM) override external minterOnly 
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L172

```
   function mint(address _target, uint256 _amount) override external minterOnly {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L179

```
   /**
    * Anyone is allowed to burn their ZCHF.
    */
   function burn(uint256 _amount) external
```

## Internal function  and private should start with underscore `_`

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L97

```
    function allowanceInternal(address owner, address spender) internal view virtual returns (uint256) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L102

```
   function allowanceInternal(address owner, address spender) internal view override returns (uint256) { 
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L37

```
    function createClone(address target) internal returns (address result) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L169

```
    function collateralBalance() internal view returns (uint256){
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L232

```
    function repayInternal(uint256 burnable) internal {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L240

```
    function notifyRepaidInternal(uint256 amount) internal {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L268

```
    function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L282

```
    function checkCollateral(uint256 collateralReserve, uint256 atPrice) internal view {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L286

```
    function emitUpdate() internal {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L67

```
    function burnInternal(address zchfHolder, address target, uint256 amount) internal {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L188

```
    function minBid(Challenge storage challenge) internal view returns (uint256) {
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L144

```
    function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal { 

```
## Unused import statement

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L5

```
    import "./IFrankencoin.sol";
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L7

```
    import "./IERC677Receiver.sol";
```

## Import statement should import specific identifiers rather than the whole file

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L7

```
    import "./ERC20.sol";
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L4

```
    import "./ERC20PermitLight.sol";
    import "./Equity.sol";  
    import "./IReserve.sol";
    import "./IFrankencoin.sol";
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L5

```
    import "./Position.sol";
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L8

```
    import "./Frankencoin.sol";
    import "./IERC677Receiver.sol"; 
    import "./ERC20PermitLight.sol";    
    import "./MathUtil.sol";
    import "./IReserve.sol";
```

## Large multiple of tens should use scientific notation

Using scientific notation for large multiple of tens will improve readability

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MathUtil.sol#L10

```
    uint256 internal constant ONE_DEC18 = 10**18;
    uint256 internal constant THRESH_DEC18 =  10000000000000000;//0.01
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L25

```
   uint256 public constant MIN_FEE = 1000 * (10**18);
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L166

```
      uint256 usableMint = (_amount * (1000_000 - _feesPPM - _reservePPM)) / 1000_000; // rounding down is fine
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L239

```
      return 1000000 * amountExcludingReserve / (1000000 - adjustedReservePPM); // 41 / (1-18%) = 50
```

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L122

```
    if (afterFees){
        return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
    } else {
        return totalMint * (1000_000 - reserveContribution) / 1000_000;
    }
```

## Compute DOMAIN_SEPERATOR only if current chainId != Initial chainId

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L61

```
    function DOMAIN_SEPARATOR() public view returns (bytes32) { 
        return
            keccak256(
                abi.encode(
                    //keccak256("EIP712Domain(uint256 chainId,address verifyingContract)");
                    bytes32(0x47e79534a245952e8b16893a336b85a3d9ea9fa8c573f3d803afb92a79469218),
                    block.chainid,
                    address(this)
                )
            );
    }
```

https://github.com/transmissions11/solmate/blob/8c0e278900fe552fa0739975bde21c6a07d84ccf/src/tokens/ERC20.sol#L159

```
    function DOMAIN_SEPARATOR() public view virtual returns (bytes32) {
        return block.chainid == INITIAL_CHAIN_ID ? INITIAL_DOMAIN_SEPARATOR : computeDomainSeparator();
    }

    function computeDomainSeparator() internal view virtual returns (bytes32) {
        return
            keccak256(
                abi.encode(
                    keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"),
                    keccak256(bytes(name)),
                    keccak256("1"),
                    block.chainid,
                    address(this)
                )
            );
    }
```

## Misleading function name

Function returns the registed minter of the position and do not check if the address is a position

https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L300

```
   function isPosition(address _position) override public view returns (address){
      return positions[_position];
   }
```