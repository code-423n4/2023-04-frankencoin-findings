## Summary 

### Gas Optimization

|no |Issue|Instances|
|----|-------|------|
| [G-01] | Use calldata instead of memory for function parameters | 4 | - | 
| [G-02] | Refactor mapping to save over 44k gas for new users that stake, 10k gas for recurring users that stake, and 10k gas for users that withdraw| 4 | - | 
| [G-03] | State variables can be cached instead of re-reading them from storage | 2 | - |
| [G-04] | Avoid emitting constants. | 19 | - |  
| [G-05] | Multiple accesses of a mapping/array should use a storage pointer |  6  | - |
| [G-06] | Change public state variable visibility to private | 11  | - |  
| [G-07] | Use multiple if statement instead of && operator   | 3   | - |   
| [G-08] | Make 3 event parameters indexed when possible   | 6  | - |  
| [G-09] | Use != 0 instead of > 0 for unsigned integer comparison   | 2  | - |  
| [G-10] | Duplicated require()/revert() Checks Should Be Refactored To A Modifier Or Function   | 1 | - |
| [G-11] | Functions guaranteed to revert when called by normal users can be marked payable   | 2 | - |
| [G-12] | Use hardcoded address instead address(this)  | 1 | - |



## [G-01] Use calldata instead of memory for function parameters
```solidity
file: Equity.sol
97    function name() external pure override returns (string memory)
112   function symbol() external pure override returns (string memory)

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L97
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L112

```solidity 
file: Frankencoin.sol
64    function name() external pure override returns (string memory) 
78    function symbol() external pure override returns (string memory)

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L64
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L68


## [G-02] Refactor mapping to save over 44k gas for new users that stake, 10k gas for recurring users that stake, and 10k gas for users that withdraw
```solidity
file:  Equity.sol
83  mapping(address => address) public delegates;

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L83

```solidity
file:     ERC20.sol
45        mapping(address => mapping(address => uint256)) private _allowances;


```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L45

```solidity
file:     Frankencoin.sol 
50        mapping(address => address) public positions;

```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L50

```solidity
file:      MintingHub.sol
37         mapping(address /** col */ => mapping(address => uint256))

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L37



## [G-03] State variables can be cached instead of re-reading them from storage

```solidity
file:    Equity.sol
190        function votes(
        address sender,
        address[] calldata helpers
    ) public view returns (uint256) {
        uint256 _votes = votes(sender);
        for (uint i = 0; i < helpers.length; i++) {
            address current = helpers[i];
            require(current != sender);
            require(canVoteFor(sender, current));
            for (uint j = i + 1; j < helpers.length; j++) {
                require(current != helpers[j]); // ensure helper unique
            }
            _votes += votes(current);
        }
        return _votes;
    }
350       function restructureCapTable(
        address[] calldata helpers,
        address[] calldata addressesToWipe
    ) public {
        require(zchf.equity() < MINIMUM_EQUITY);
        checkQualified(msg.sender, helpers);
        for (uint256 i = 0; i < addressesToWipe.length; i++) {
            address current = addressesToWipe[0];
            _burn(current, balanceOf(current));
        }
    }

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L190-L202

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L309-L316

## [G-04] Avoid emitting constants.
```solidity
file:     Equity.sol
280       emit Trade(msg.sender, int(shares), amount, price());

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L280

```solidity
file:     ERC20.sol
158       emit Transfer(sender, recipient, amount);
186       emit Transfer(address(0), recipient, amount);
205       emit Transfer(account, address(0), amount);
223       emit Approval(owner, spender, value);

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L158
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L186
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L205
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L223

```solidity
file:   Frankencoin.sol
89           emit MinterApplied(
            _minter,
            _applicationPeriod,
            _applicationFee,
            _message
        );
156     emit MinterDenied(_minter, _message);        

```

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L89
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L156

```solidity
file: 
146     emit ChallengeStarted(
            msg.sender,
            address(position),
            _collateralAmount,
            pos
        );
176           emit ChallengeStarted(
            challenge.challenger,
            address(challenge.position),
            challenge.size,
            _challengeNumber
        ); 
177      emit ChallengeStarted(
            copy.challenger,
            address(copy.position),
            copy.size,
            pos
        );
206     emit NewBid(_challengeNumber, _bidAmountZCHF, msg.sender);

212     emit ChallengeAverted(
                address(challenge.position),
                _challengeNumber
            );
274        emit ChallengeSucceeded(
            address(challenge.position),
            challenge.bid,
            _challengeNumber
        );
292     emit PostPonedReturn(
                collateral,
                challenge.challenger,
                challenge.size
            );        

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L146
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L176
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L177
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L206
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L212
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L274
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L292

```solidity
file:  
42    emit OwnershipTransferred(oldOwner, newOwner);

``` 
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L42

```solidity
file:    Position.sol
69       emit PositionOpened(_owner, original, _zchf, address(collateral), _liqPrice);
85       emit PositionOpened(owner, original, address(zchf), address(collateral), 
_price);
113      emit PositionDenied(msg.sender, message);
287      emit MintingUpdate(collateralBalance(), price, minted, limit);

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L69
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L85
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L113
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L287


## [G-05] Multiple accesses of a mapping/array should use a storage pointer
```solidity
file:   Equity.sol 
83      mapping(address => address) public delegates;
205      function votes(
        address sender,
        address[] calldata helpers
    ) public view returns (uint256) {
        uint256 _votes = votes(sender);
        for (uint i = 0; i < helpers.length; i++) {
            address current = helpers[i];
            require(current != sender);
            require(canVoteFor(sender, current));
            for (uint j = i + 1; j < helpers.length; j++) {
                require(current != helpers[j]); // ensure helper unique
            }
            _votes += votes(current);
        }
        return _votes;
    }

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L83
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L190-L202
```solidity
file:    ERC20.sol
43       mapping(address => uint256) private _balances;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#L43
```solidity
file:      Frankencoin.sol
50         mapping(address => address) public positions;

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L50
```solidity
file:   MintingHub.sol
31      Challenge[] public challenges;
37      mapping(address /** col */ => mapping(address => uint256))
        public pendingReturns;
     

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L31
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L37

## [G-06]  Change public state variable visibility to private
```solidity
file:   Frankencoin.sol
26      uint256 public immutable MIN_APPLICATION_PERIOD
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L26

```solidity
file:   Ownable.sol
21      address public owner;
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol#L21
```solidity
file:    Position.sol
21       uint256 public price;
22       uint256 public minted
23       uint256 public challengedAmount
26       uint256 public cooldown
27       uint256 public limit
29       uint256 public immutable start
30       uint256 public immutable expiration;
32       address public immutable original
33       address public immutable hub; 

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L21
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L22
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L23
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L26
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L27
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L29
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L30
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L32
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L33
 
## [G-07]  Use multiple if statement instead of && operator
```solidity
file:    Frankencoin.sol
84       if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0)
85       if (_applicationFee < MIN_FEE && totalSupply() > 0) revert FeeTooLow();
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L84
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L85

```solidity
file:   Position.sol
294     if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L294

## [G-08]  Make 3 event parameters indexed when possible
```solidity
file:   Frankencoin.sol
53      event MinterDenied(address indexed minter, string message);

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L53

```solidity
file:   MintingHub.sol
49      event ChallengeAverted(address indexed position, uint256 number);
50      event ChallengeSucceeded(
        address indexed position,
        uint256 bid,
        uint256 number
    );
51      event NewBid(uint256 challengedId, uint256 bidAmount, address bidder);
52      event PostPonedReturn(
        address collateral,
        address indexed beneficiary,
        uint256 amount
    );


``` 
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L49
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L50
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L51
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L52

```solidity
file:   Position.sol
43      event PositionDenied(address indexed sender, string message);

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L43

## [G-09]   Use != 0 instead of > 0 for unsigned integer comparison
```solidity
file:   Equity.sol 
114     if (amount > 0) 
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L114
```solidity
file:    Frankencoin.sol
104      if (explicit > 0) 
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L104

### [G-10] Duplicated require()/revert() Checks Should Be Refactored To A Modifier Or Function
### NOT: this findings output is missed by Automated and i found here 

```solidity
file:  ERC20PermitLight.sol
56     require(
               recoveredAddress != address(0) && recoveredAddress == owner,
                "INVALID_SIGNER"
            );

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol#L56

### [G-11] Functions guaranteed to revert when called by normal users can be marked payable 
### NOT: this findings output is missed by Automated and i found here 

```solidity
file:  Frankencoin.sol
172    function mint(address _target, uint256 _amount) override external minterOnly { 
262      function burn(address _owner, uint256 _amount) override external minterOnly {
```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L172
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L262

### [G-12] Use hardcoded address instead address(this)  
### NOT: this findings output is missed by Automated and i found here 
```solidity
file: MintingHub.sol  
116   require(zchf.isPosition(position) == address(this), "not our pos");

```
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L116

