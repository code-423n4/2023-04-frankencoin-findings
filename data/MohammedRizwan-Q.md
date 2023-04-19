### Low risk issues
| |Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | For immutable variables, Zero address and zero value checks are missing in constructor | 10 |
| [L&#x2011;02] | Withdraw to zero address should be avoided | 1 |

### Non-Critical issues
| |Issue|Instances| |
|-|:-|:-:|:-:|
| [N&#x2011;01] | For internal functions, follow Solidity standard naming conventions | 21 |
| [N&#x2011;02] | Solidity compiler version should be exactly same in all smart contracts | 10 |
| [N&#x2011;03] | Use named parameters for mapping type declarations | 8 |
| [N&#x2011;04] | Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18) | 4 |

### Low risk issues
### [L&#x2011;01]  For immutable variables, Zero address and zero value checks are missing in constructor
Zero address and zero value checks should be used in the constructors, to avoid the risk of setting a storage variable as zero address and zero value at the time of deployment.
There are 10 instances of this issue.

In Position.sol contract constructor, Below are immutable variables.

```solidity
File: contracts/Position.sol

24    uint256 public immutable challengePeriod; // challenge period in seconds
---

29    uint256 public immutable start; // timestamp when minting can start
30    uint256 public immutable expiration; // timestamp at which the position expires
31
32    address public immutable original; // originals point to themselves, clone to their origin
33    address public immutable hub; // the hub this position was created by
34    IFrankencoin public immutable zchf; // currency
35    IERC20 public override immutable collateral; // collateral
36    uint256 public override immutable minimumCollateral; // prevent dust amounts
37
38    uint32 public immutable mintingFeePPM;
39    uint32 public immutable reserveContribution; // in ppm
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L24-L39)

```solidity
File: contracts/Position.sol

50    constructor(address _owner, address _hub, address _zchf, address _collateral, 
51        uint256 _minCollateral, uint256 _initialLimit, uint256 initPeriod, uint256 _duration,
52        uint256 _challengePeriod, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) {
53        require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values
54        setOwner(_owner);
55        original = address(this);
56        hub = _hub;
57        price = _liqPrice;
58        zchf = IFrankencoin(_zchf);
59        collateral = IERC20(_collateral);
60        mintingFeePPM = _mintingFeePPM;
61        reserveContribution = _reservePPM;
62        minimumCollateral = _minCollateral;
63        challengePeriod = _challengePeriod;
64        start = block.timestamp + initPeriod; // one week time to deny the position
65        cooldown = start;
66        expiration = start + _duration;
67        limit = _initialLimit;
68        
69        emit PositionOpened(_owner, original, _zchf, address(collateral), _liqPrice);
70    }
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L50-L70)

### Recommended Mitigation Steps:
Immutable variables once set during deployment can not be changed, It is better not to take risk and do the address and value validation so that error should not happen. Below is the recommended code.

```solidity
File: contracts/Position.sol

    constructor(address _owner, address _hub, address _zchf, address _collateral, 
        uint256 _minCollateral, uint256 _initialLimit, uint256 initPeriod, uint256 _duration,
        uint256 _challengePeriod, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) {
+       require(_hub != address(0), "invalid address");
+       require(_zchf != address(0), "invalid address");
+       require(_collateral != address(0), "invalid address");
+       require(_minCollateral != 0, "invalid amount");
+       require(_challengePeriod != 0, "invalid period");
+       require(_mintingFeePPM != 0, "invalid fee");
+       require(_reservePPM != 0, "invalid amount");
        require(initPeriod >= 3 days); // must be at least three days, recommended to use higher values
        setOwner(_owner);
        original = address(this);
        hub = _hub;
        price = _liqPrice;
        zchf = IFrankencoin(_zchf);
        collateral = IERC20(_collateral);
        mintingFeePPM = _mintingFeePPM;
        reserveContribution = _reservePPM;
        minimumCollateral = _minCollateral;
        challengePeriod = _challengePeriod;
        start = block.timestamp + initPeriod; // one week time to deny the position
        cooldown = start;
        expiration = start + _duration;
        limit = _initialLimit;
        
        emit PositionOpened(_owner, original, _zchf, address(collateral), _liqPrice);
    }
    }
```

In MintingHub.sol contract constructor, Below are immutable variables.

```solidity
File: contracts/MintingHub.sol

28    IPositionFactory private immutable POSITION_FACTORY; // position contract to clone
---

30    IFrankencoin public immutable zchf; // currency
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L28-L30)

```solidity
File: contracts/MintingHub.sol

54    constructor(address _zchf, address factory) {
55        zchf = IFrankencoin(_zchf);
56        POSITION_FACTORY = IPositionFactory(factory);
57    }
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L54-L57)


### Recommended Mitigation Steps:
Immutable variables once set during deployment can not be changed, It is better not to take risk and do the address and value validation so that error should not happen. Below is the recommended code.

```solidity
File: contracts/MintingHub.sol

    constructor(address _zchf, address factory) {
+       require(_zchf != address(0), "invalid address");
+       require(factory != address(0), "invalid address");
        zchf = IFrankencoin(_zchf);
        POSITION_FACTORY = IPositionFactory(factory);
    }
```

In Frankencoin.sol contract constructor, Below are immutable variables.
```solidity
File: contracts/Frankencoin.sol

26   uint256 public immutable MIN_APPLICATION_PERIOD; // for example 10 days
---

31   IReserve override public immutable reserve;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L26-L31)

```solidity
File: contracts/Frankencoin.sol

59   constructor(uint256 _minApplicationPeriod) ERC20(18){
60      MIN_APPLICATION_PERIOD = _minApplicationPeriod;
61      reserve = new Equity(this);
62   }
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L59-L62)

### Recommended Mitigation steps

```solidity
File: contracts/Frankencoin.sol

   constructor(uint256 _minApplicationPeriod) ERC20(18){
+     require(_minApplicationPeriod != 0, "invalid period");
      MIN_APPLICATION_PERIOD = _minApplicationPeriod;
      reserve = new Equity(this);
   }
```

### [L&#x2011;02]  Withdraw to zero address should be avoided
Withdraw collateral or tokens must be avoided to address(0) as if mistakenly sent to address(0), it can not be recovered. withdraw() can be accessed by onlyOwner but address(0) transfer can not be rule out as onlyOwner can also transfer by mistake to address(0).
There is 1 instance of this issue.

```solidity
File: contracts/Position.sol

249    function withdraw(address token, address target, uint256 amount) external onlyOwner {
250        if (token == address(collateral)){
251            withdrawCollateral(target, amount);
252        } else {
253            IERC20(token).transfer(target, amount);
254        }
255    }
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L249-L255)

### Recommended Mitigation steps
Add zero address check validation in withdraw function to prevent this issue.

```solidity
File: contracts/Position.sol

    function withdraw(address token, address target, uint256 amount) external onlyOwner {
+       require(target != address(0), "invalid address");
        if (token == address(collateral)){
            withdrawCollateral(target, amount);
        } else {
            IERC20(token).transfer(target, amount);
        }
    }
```

### Non-Critical issues
### [N&#x2011;01]  For internal functions, follow Solidity standard naming conventions
Proper use of _ as a function name prefix should be taken care and a common pattern is to prefix internal and private function names with _.
This is partially incorporated in contracts. Below internal functions does not follow this pattern which leads to confusion while reading code and affects overall readability of the code.

There are 21 instances of this issue:

```solidity
File: contracts/Position.sol

169    function collateralBalance() internal view returns (uint256){
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L169)

```solidity
File: contracts/Position.sol

193    function mintInternal(address target, uint256 amount, uint256 collateral_) internal {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L193)

```solidity
File: contracts/Position.sol

202    function restrictMinting(uint256 period) internal {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L202)

```solidity
File: contracts/Position.sol

232    function repayInternal(uint256 burnable) internal {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L232)

```solidity
File: contracts/Position.sol

240    function notifyRepaidInternal(uint256 amount) internal {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L240)

```solidity
File: contracts/Position.sol

268    function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L268)

```solidity
File: contracts/Position.sol

282    function checkCollateral(uint256 collateralReserve, uint256 atPrice) internal view {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L282)

```solidity
File: contracts/Position.sol

286    function emitUpdate() internal {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L286)

```solidity
File: contracts/MintingHub.sol

188    function minBid(Challenge storage challenge) internal view returns (uint256) {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L188)

```solidity
File: contracts/MintingHub.sol

287    function returnCollateral(Challenge storage challenge, bool postpone) internal {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L287)

```solidity
File: contracts/Equity.sol

144    function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L144)

```solidity
File: contracts/Equity.sol

157    function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L157)

```solidity
File: contracts/Equity.sol

172    function anchorTime() internal view returns (uint64){
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L172)

```solidity
File: contracts/Equity.sol

225    function canVoteFor(address delegate, address owner) internal view returns (bool) {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L225)

```solidity
File: contracts/Equity.sol

266    function calculateSharesInternal(uint256 capitalBefore, uint256 investment) internal view returns (uint256) {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L266)

```solidity
File: contracts/Frankencoin.sol

102   function allowanceInternal(address owner, address spender) internal view override returns (uint256) {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L102)

```solidity
File: contracts/ERC20.sol

97    function allowanceInternal(address owner, address spender) internal view virtual returns (uint256) {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L97)

```solidity
File: contracts/Ownable.sol

39    function setOwner(address newOwner) internal {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L39)

```solidity
File: contracts/PositionFactory.sol

37    function createClone(address target) internal returns (address result) {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L37)

```solidity
File: contracts/StablecoinBridge.sol

49    function mintInternal(address target, uint256 amount) internal {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L49)

```solidity
File: contracts/StablecoinBridge.sol

67    function burnInternal(address zchfHolder, address target, uint256 amount) internal {
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L67)

### [N&#x2011;02]  Solidity compiler version should be exactly same in all smart contracts
Solidity compiler version should be exactly same in all smart contracts. Different Solidity compiler versions are used, the following contracts have mix versions. 

There are 10 instances of this issue.

```solidity
File: contracts/Position.sol

2    pragma solidity ^0.8.0;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L2)

```solidity
File: contracts/Equity.sol

4    pragma solidity >=0.8.0 <0.9.0;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L4)

```solidity
File: contracts/ERC20.sol

14   pragma solidity ^0.8.0;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L12)

```solidity
File: contracts/MathUtil.sol

3    pragma solidity >=0.8.0 <0.9.0;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MathUtil.sol#L3)

```solidity
File: contracts/ERC20PermitLight.sol

5    pragma solidity ^0.8.0;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L5)

```solidity
File: contracts/Frankencoin.sol

2    pragma solidity ^0.8.0;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L2)

```solidity
File: contracts/MintingHub.sol

2    pragma solidity ^0.8.0;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L2)

```solidity
File: contracts/StablecoinBridge.sol

2    pragma solidity ^0.8.0;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L2)

```solidity
File: contracts/PositionFactory.sol

2    pragma solidity ^0.8.0;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L2)

```solidity
File: contracts/Ownable.sol

9    pragma solidity ^0.8.0;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L9)

### Recommendation Mitigation steps
Versions must be consistent with each other.




### [N&#x2011;03]  Use named parameters for mapping type declarations
Consider using named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability. This feature is present since Solidity 0.8.18.
There are 8 instances of this issue.

```solidity
File: contracts/MintingHub.sol

37    mapping (address /** col */ => mapping (address => uint256)) public pendingReturns;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L37)

```solidity
File: contracts/Equity.sol

83    mapping (address => address) public delegates;
---

88    mapping (address => uint64) private voteAnchor; // 40 Bit for the block number, 24 Bit sub-block time resolution
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L83-L88)

```solidity
File: contracts/Frankencoin.sol

45   mapping (address => uint256) public minters;
---

50   mapping (address => address) public positions;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L45-L50)

```solidity
File: contracts/ERC20.sol

43    mapping (address => uint256) private _balances;
44
45    mapping (address => mapping (address => uint256)) private _allowances;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L43-L45)

```solidity
File: contracts/ERC20PermitLight.sol

15    mapping(address => uint256) public nonces;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L15)


### [N&#x2011;04]  Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)
While the compiler knows to optimize away the exponentiation, it’s still better coding practice to use idioms that do not require compiler optimization, if they exist
There are 4 instances of this issue:

```solidity
File: contracts/Equity.sol

253        require(totalSupply() < 2**128, "total supply exceeded");
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L253)

```solidity
File: contracts/Frankencoin.sol

25     uint256 public constant MIN_FEE = 1000 * (10**18);
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L25)

```solidity
File: contracts/MathUtil.sol

10    uint256 internal constant ONE_DEC18 = 10**18;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MathUtil.sol#L10)

```solidity
File: contracts/MintingHub.sol

20    uint256 public constant OPENING_FEE = 1000 * 10**18;
```
[Link to code](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L20)
