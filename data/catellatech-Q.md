`Dear Frankencoin team, as we have gone through each contract within the scope, we have noticed good practices that have been implemented. However, we have identified some inconsistencies that we recommend addressing. We understand that every team has a different level of good practices, but we believe that at least 90% of the recommendations in the following report should be applied for better gas efficiency, readability, and most importantly, safety.`

`Note: We have provided a description of the situation and recommendations to follow, including articles and resources we have created to help identify the problem and address it quickly, and to implement them in future projects.`


### Issues
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| S | Suggestions | Suggestion Details |

| Total Found Issues | 21 |
|:--:|:--:|

### Low Risk
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [L-01] | ASSEMBLY CODES SPECIFIC - SHOULD HAVE COMMENTS | 1 |
| [L-02] | MISSING EMERGENCY STOP (CIRCUIT BREAKER) PATTERN | 1 |
| [L-03] | INCONSISTENT SOLIDITY PRAGMA | 2 |
| [L-04] | A SINGLE POINT OF FAILURE | 6 |
| [L-05] | MISSING EVENTS FOR ONLY FUNCTIONS THAT CHANGE CRITICAL PARAMETERS | 9 |
| [L-06] | USE SMTChecker | All contracts |
| [L-07] | FUNCTION OVERLOADING | 16 |

| Total Low Issues | 7 | Total Instances | 35 |
|:--:|:--:|:--:|--:|

### Non-Critical
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-01] | FUNCTIONS DO NOT FOLLOW THE BEST PRACTICES OF SOLIDITY | 5 |
| [N-02] | MAPPINGS DO NOT FOLLOW THE STYLE OF SOLIDITY | 7 |
| [N-03] | TAKE ADVANTAGE OF CUSTOM ERROR'S RETURN VALUE PROPERTY | 17 |
| [N-04] | SORT SOLIDITY OPERATIONS USING SHORT-CIRCUIT MODE | 7 |
| [N-05] | CONTRACT DOES NOT FOLLOW THE SOLIDITY STYLE GUIDE'S SUGGESTED LAYOUT ORDERING | 3 |
| [N-06] | THE MODIFIER ORDER FOR A FUNCTION SHOULD BE | 15 |
| [N-07] | USING WHILE FOR UNBOUNDED LOOPS ISN'T RECOMMENDED | 1 |
| [N-08] | ALSO VALUES IN ASSEMBLY CAN BE STORED IN CONSTANT VARIABLES | 1 |
| [N-09] | CONSTANTS ON THE LEFT ARE BETTER | 1 |

| Total Non-Critical Issues | 9 | Total Instances | 57 |
|:--:|:--:|:--:|--:|

### Refactor Issues 
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [R-01] | SHORTHAND WAY TO WRITE IF/ELSE STATEMENT | 9 |
| [R-02] | FUNCTION NAMING SUGGESTIONS | 21 |
| [R-03] | AVOID BEING REDUNDANT AND STORE VALUES IN CONSTANT VARIABLES | 4 |

| Total Refactor Issues | 3 | Total Instances | 34 |
|:--:|:--:|:--:|--:|


### Suggestion Details 
| Count | Explanation | 
|:--:|:-------|
| [S-01] | WE SUGGEST USING THE OPENZEPPELIN LIBRARIES |  | 
| [S-02] | WE SUGGEST TO USE NAMED PARAMETERS FOR MAPPING TYPE DECLARATIONS |  | 

| Total Suggestions | 2 |
|:--:|:--:|

# Detailed Findings

## [L-01] ASSEMBLY CODES SPECIFIC - SHOULD HAVE COMMENTS
Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Aseembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

### PROOF OF CONCEPT
```solidity
main/contracts/PositionFactory.sol
    // @audit Add clear documentation in assembly code.
        39: assembly {
                    let clone := mload(0x40)
                    mstore(clone, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
                    mstore(add(clone, 0x14), targetBytes)
                    mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
                    result := create(0, clone, 0x37)
                }
```

### RECOMMENDATION
It is essential to clearly and comprehensively document all activities related to critical function `assembly` within the project. By doing so, a more complete and accurate understanding of what is being done is provided, which is important both for auditors and for long-term project maintenance. By following the practices of monitoring and controlling project work, it can be ensured that all assemblies are properly documented in accordance with the project's objectives and goals.


## [L-02] MISSING EMERGENCY STOP (CIRCUIT BREAKER) PATTERN
At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an `EMERGENCY STOP (CIRCUIT BREAKER) PATTERN`. Use the follow example to implement it into in the `MintingHub.sol`, in the following functions `openPosition`, `returnPostponedCollateral`.

[Emergency Stop Pattern Example](https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol)


## [L-03] INCONSISTENT SOLIDITY PRAGMA
The source files have different solidity compiler ranges referenced. This leads to potential security flaws between deployed contracts depending on the compiler version chosen for any particular file. It also greatly increases the cost of maintenance as different compiler versions have different semantics and behavior.

### PROOF OF CONCEPT
```solidity
main/contracts/MintingHub.sol
    2: pragma solidity ^0.8.0;

main/contracts/Equity.sol
    4: pragma solidity >=0.8.0 <0.9.0; // @audit One more contract has this pragma version 
```
### MITIGATION
We recommend to fix a definite compiler range that is consistent between contracts and upgrade any affected contracts to conform to the specified compiler.


## [L-04] A SINGLE POINT OF FAILURE
The `owner` role has a single point of failure and `onlyOwner` can use critical a few functions.

owner role in the project:

- Owner is not behind a multisig and changes are not behind a timelock.

Even if protocol admins/developers are not malicious there is still a chance for `Owner` keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

### PROOF OF CONCEPT
```solidity
main/contracts/Position.sol
    8: import "./Ownable.sol";
    14: contract Position is Ownable, IPosition, MathUtil {
```

Functions that have the onlyOwner modifier: 
```solidity
main/contracts/Position.sol
    132: function adjust(uint256 newMinted, uint256 newCollateral, uint256 newPrice) public onlyOwner {
    159: function adjustPrice(uint256 newPrice) public onlyOwner noChallenge {
    177: function mint(address target, uint256 amount) public onlyOwner noChallenge noCooldown alive {
    227: function repay(uint256 amount) public onlyOwner {
    249: function withdraw(address token, address target, uint256 amount) external onlyOwner {
    263: function withdrawCollateral(address target, uint256 amount) public onlyOwner noChallenge noCooldown {                   
```
This increases the risk of `A single point of failure`.

### MITIGATION
Add a time lock to critical functions. Admin-only functions that change critical parameters should emit events and have timelocks.

Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services.

Allow only multi-signature wallets to call the function to reduce the likelihood of an attack.

https://twitter.com/danielvf/status/1572963475101556738?s=20&t=V1kvzfJlsx-D2hfnG0OmuQ

Also detail them in documentation and NatSpec comments.


## [L-05] MISSING EVENTS FOR ONLY FUNCTIONS THAT CHANGE CRITICAL PARAMETERS
There are functions that change critical parameters should emit events. Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services. The alternative of directly querying on-chain contract state for such changes is not considered practical for most users/usages.

missing events and timelocks do not promote transparency and if such changes immediately affect users' perception of fairness or trustworthiness, they could exit the protocol causing a reduction in liquidity which could negatively impact protocol TVL and reputation.

### PROOF OF CONCEPT
```solidity
// @audit Add events to the following functios
    main/contracts/Position.sol
        249: function withdraw(address token, address target, uint256 amount) external onlyOwner {
        292: function notifyChallengeStarted(uint256 size) external onlyHub {
        329: function notifyChallengeSucceeded(address _bidder, uint256 _bid, uint256 _size) external onlyHub returns (address, uint256, uint256, uint256, uint32) {

    main/contracts/MintingHub.sol
        281: function returnPostponedCollateral(address collateral, address target) external {

    /main/contracts/Equity.sol
        186: function totalVotes() public view returns (uint256) {

    main/contracts/Frankencoin.sol
        280: function notifyLoss(uint256 _amount) override external minterOnly {

    main/contracts/StablecoinBridge.sol
        44: function mint(address target, uint256 amount) public {
        63: function burn(address target, uint256 amount) external {
        75: function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool){
```
### MITIGATION
Add Event-Emit, for example in these cases:
```diff
+ emit SomeEvent(address to, uint amount);
```

## [L-06] USE SMTChecker
The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

[ Check the next example. ](https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19)


## [L-07] FUNCTION OVERLOADING
Having multiple functions with the same name in a smart contract can be dangerous or not a good practice for several reasons:

- Confusion: If there are several functions with the same name, it can be confusing for developers and users who are interacting with the smart contract. This can lead to errors and misunderstandings in the use of the contract.

- Security vulnerabilities: If multiple functions are defined with the same name, attackers can attempt to exploit this vulnerability to access or modify data or functionalities of the smart contract.

- Network overload: If there are multiple functions with the same name, there may be an impact on the efficiency and speed of the contract, as the network may be confused in trying to determine which function should be executed.

See more about this on [Solidity Docs.](https://docs.soliditylang.org/en/v0.8.19/contracts.html#function-overloading)

### PROOF OF CONCEPT
```solidity
main/contracts/MintingHub.sol
59: function openPosition(
88: function openPosition(
181: function minBid(uint256 challenge) public view returns (uint256) {
188: function minBid(Challenge storage challenge) internal view returns (uint256) {
235: function end(uint256 _challengeNumber) external {
252: function end(uint256 _challengeNumber, bool postponeCollateralReturn) public {

main/contracts/Equity.sol
127: function canRedeem() external view returns (bool)
135: function canRedeem(address owner) public view returns (bool)
179: function votes(address holder) public view returns (uint256)
190: function votes(address sender, address[] calldata helpers) public view returns (uint256)

main/contracts/Frankencoin.sol
165: function mint(address _target, uint256 _amount, uint32 _reservePPM, uint32 _feesPPM) override external minterOnly
172: function mint(address _target, uint256 _amount) override external minterOnly 

main/contracts/StablecoinBridge.sol
36: function mint(uint256 amount) external 
44: function mint(address target, uint256 amount) public
55: function burn(uint256 amount) external 
63: function burn(address target, uint256 amount) external
```


## [N-01] FUNCTIONS DO NOT FOLLOW THE BEST PRACTICES OF SOLIDITY
In many functions, they do not follow the best practices patterns recommended by the [Solidity documentation](https://docs.soliditylang.org/en/v0.8.19/style-guide.html#function-declaration), making these changes would improve the readability of the code.

### PROOF OF CONCEPT
```solidity
main/contracts/Position.sol
50: constructor(address _owner, address _hub, address _zchf, address _collateral, 
        uint256 _minCollateral, uint256 _initialLimit, uint256 initPeriod, uint256 _duration,
        uint256 _challengePeriod, uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM)
329: function notifyChallengeSucceeded(address _bidder, uint256 _bid, uint256 _size) external onlyHub returns (address, uint256, uint256, uint256, uint32)

main/contracts/MintingHub.sol
59: function openPosition(
        address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
        uint256 _mintingMaximum, uint256 _expirationSeconds, uint256 _challengeSeconds,
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address)
88: function openPosition(
        address _collateralAddress, uint256 _minCollateral, uint256 _initialCollateral,
        uint256 _mintingMaximum, uint256 _initPeriodSeconds, uint256 _expirationSeconds, uint256 _challengeSeconds,
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reservePPM) public returns (address)

main/contracts/PositionFactory.sol
13: function createNewPosition(address _owner, address _zchf, address _collateral, uint256 _minCollateral, 
        uint256 _initialLimit, uint256 initPeriod, uint256 _duration, uint256 _challengePeriod, 
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reserve) 
        external returns (address)
```

### MITIGATION
We recommend following your own best practices as applied in the ERC20PermitLight.sol contract on line 21.


## [N-02] MAPPINGS DO NOT FOLLOW THE STYLE OF SOLIDITY
The [solidity documentation](https://docs.soliditylang.org/en/v0.8.19/style-guide.html#mappings) strongly recommends the following: 
`In variable declarations, do not separate the keyword mapping from its type by a space. Do not separate any nested mapping keyword from its type by whitespace.`

### PROOF OF CONCEPT
```solidity
main/contracts/MintingHub.sol
    37: mapping (address /** col */ => mapping (address => uint256)) public pendingReturns;

main/contracts/Equity.sol
    83: mapping (address => address) public delegates;
    88: mapping (address => uint64) private voteAnchor;

main/contracts/Frankencoin.sol
    45: mapping (address => uint256) public minters;
    50: mapping (address => address) public positions;

main/contracts/ERC20.sol
    43: mapping (address => uint256) private _balances;
    45: mapping (address => mapping (address => uint256)) private _allowances;
```

### MITIGATION
We recommend following your own best practices as applied in the ERC20PermitLight.sol contract on line 15.


## [N-03] TAKE ADVANTAGE OF CUSTOM ERROR'S RETURN VALUE PROPERTY
An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the `()` sign, this kind of approach provides a serious advantage in debugging and examining the revert details of dapps such as tenderly.

### PROOF OF CONCEPT
```solidity
main/contracts/Position.sol 
    110: if (block.timestamp >= start) revert TooLate();
    194: if (minted + amount > limit) revert LimitExceeded();
    283: if (collateralReserve * atPrice < minted * ONE_DEC18) revert InsufficientCollateral();
    294: if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();
    367: if (block.timestamp > expiration) revert Expired();
    374: if (block.timestamp <= cooldown) revert Hot();
    381: if (challengedAmount > 0) revert Challenged();
    388: if (msg.sender != address(hub)) revert NotHub();

main/contracts/MintingHub.sol
    201: if (block.timestamp >= challenge.end) revert TooLate();
    202: if (expectedSize != challenge.size) revert UnexpectedSize();

main/contracts/Equity.sol
    211: if (_votes * 10000 < QUORUM * totalVotes()) revert NotQualified();

main/contracts/Frankencoin.sol
    84: if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
    85: if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
    86: if (minters[_minter] != 0) revert AlreadyRegistered();
    126: if (!isMinter(msg.sender)) revert NotMinter();
    153: if (block.timestamp > minters[_minter]) revert TooLate();
    267: if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();
```


## [N-04] SORT SOLIDITY OPERATIONS USING SHORT-CIRCUIT MODE
Short-circuiting is a solidity contract development model that uses `OR/AND` logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

### PROOF OF CONCEPT
```solidity
/main/contracts/Position.sol
    294: if (size < minimumCollateral && size < collateralBalance()) revert ChallengeTooSmall();

/main/contracts/Frankencoin.sol
    84: if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
    85: if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();
    106: else if (isMinter(spender) || isMinter(isPosition(spender))){
    265: if (!isMinter(msg.sender) && !isMinter(positions[msg.sender])) revert NotMinter();
    294: return minters[_minter] != 0 && block.timestamp >= minters[_minter];

main/contracts/ERC20PermitLight.sol
    56: require(recoveredAddress != address(0) && recoveredAddress == owner, "INVALID_SIGNER");
```
### RECOMMENDATION
```solidity
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 

//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```

## [N-05] CONTRACT DOES NOT FOLLOW THE SOLIDITY STYLE GUIDE'S SUGGESTED LAYOUT ORDERING
The [style guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout) says that, within a contract, the ordering should be: 
- 1. Type declarations 
- 2. State variables,
- 3. Events 
- 4. Modifiers 
- 5. Functions 
but the contract(s) below do not follow this ordering.

- [Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol)
- [MintingHub.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol)
- [Ownable.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Ownable.sol)


## [N-06] THE MODIFIER ORDER FOR A FUNCTION SHOULD BE
The following functions do not follow the styles established by Solidity.
- Visibility
- Mutability
- Virtual
- Override
- Custom modifiers

YES:
```solidity
uint8 public immutable override decimals;

function totalSupply() public view override returns (uint256) {
        return _totalSupply;
    }
```
NO:
```solidity
IReserve override public immutable reserve;

function name() override external pure returns (string memory) {
        return "Frankencoin Pool Share";
    }
```
### PROOF OF CONCEPT
```solidity
// @audit The `override` should not be in that position in the functions.
main/contracts/Equity.sol
    97: function name() override external pure returns (string memory) {
    101: function symbol() override external pure returns (string memory) {
    112: function _beforeTokenTransfer(address from, address to, uint256 amount) override internal {

main/contracts/Frankencoin.sol
    31: IReserve override public immutable reserve;
    64: function name() override external pure returns (string memory){
    68: function symbol() override external pure returns (string memory){
    83: function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {
    125: function registerPosition(address _position) override external {
    152: function denyMinter(address _minter, address[] calldata _helpers, string calldata _message) override external {
    165: function mint(address _target, uint256 _amount, uint32 _reservePPM, uint32 _feesPPM) override external minterOnly {
    172: function mint(address _target, uint256 _amount) override external minterOnly {
    262: function burn(address _owner, uint256 _amount) override external minterOnly {
    280: function notifyLoss(uint256 _amount) override external minterOnly {
    293: function isMinter(address _minter) override public view returns (bool){
    300: function isPosition(address _position) override public view returns (address){
```
### RECOMMENDATION
We recommend following your own best practices as applied in the whole contract [ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol).


## [N-07] USING WHILE FOR UNBOUNDED LOOPS ISN'T RECOMMENDED
Improve the efficiency and stability of your code by avoiding unbounded `while loops`. Instead of using a `while loop` for unbounded iterations, it is recommended to use other loop structures like `for` that have a clearer structure and can provide better control flow for the loop. Don't write loops that are unbounded as this can hit the gas limit, causing your transaction to fail. For the reason above, while and do-while loops are rarely used.

### PROOF OF CONCEPT
```solidity
main/contracts/MathUtil.sol
// @audit Avoid unbounded loops
22:   do {
            xOld = x;
            uint256 powX3 = _mulD18(_mulD18(x, x), x);
            x = _mulD18(x, _divD18( (powX3 + 2 * _v) , (2 * powX3 + _v)));
            cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
        } while ( cond );

```
### RECOMMENDATION
- Avoid unbounded loops: As mentioned before, it is important to avoid unbounded while loops in your smart contracts. If you need to make a loop, make sure that the number of iterations is limited and known in advance.


## [N-08] ALSO VALUES IN ASSEMBLY CAN BE STORED IN CONSTANT VARIABLES
Values in assembly can also benefit from constant variables, which would also increase the readability of these functions

### PROOF OF CONCEPT
```solidity
main/contracts/PositionFactory.sol
39: assembly {
        let clone := mload(0x40)
        mstore(clone, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
        mstore(add(clone, 0x14), targetBytes)
        mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
        result := create(0, clone, 0x37)
    }
```

## [N-09] CONSTANTS ON THE LEFT ARE BETTER
If you use the constant first you support structures that veil programming errors. And one should only produce code either to add functionality, fix an programming error or trying to support structures to avoid programming errors (like design patterns). [See this](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html)

### PROOF OF CONCEPT
```solidity
main/contracts/Frankencoin.sol
85: if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow();

main/contracts/ERC20.sol
128: if (currentAllowance < INFINITY)
```

## [R-01] SHORTHAND WAY TO WRITE IF/ELSE STATEMENT
The normal if/else statement can be refactored using a shorthand syntax which:

- Improves readability.
- Shortens the overall SLOC (source lines of code)."

### PROOF OF CONCEPT
```solidity
main/contracts/Position.sol
    160: if (newPrice > price) {
                restrictMinting(3 days);
            } else {
                checkCollateral(collateralBalance(), newPrice);
            }
            price = newPrice;
            emitUpdate();
        }

    184: if (time >= exp){
                return 0;
            } else {
                return uint32(mintingFeePPM - mintingFeePPM * (time - start) / (exp - start));
            }

    250: if (token == address(collateral)){
                withdrawCollateral(target, amount);
            } else {
                IERC20(token).transfer(target, amount);
            }

main/contracts/MintingHub.sol
    288: if (postpone){
            // Postponing helps in case the challenger was blacklisted on the collateral token or otherwise cannot receive it at the moment.
            address collateral = address(challenge.position.collateral());
            pendingReturns[collateral][challenge.challenger] += challenge.size;
            emit PostPonedReturn(collateral, challenge.challenger, challenge.size);
        } else {
            challenge.position.collateral().transfer(challenge.challenger, challenge.size); // return the challenger's collateral
        }

main/contracts/Equity.sol
    158: if (to != address(0x0)) {
                uint256 recipientVotes = votes(to); // for example 21 if 7 shares were held for 3 blocks
                uint256 newbalance = balanceOf(to) + amount; // for example 11 if 4 shares are added
                voteAnchor[to] = uint64(anchorTime() - recipientVotes / newbalance); // new example anchor is only 21 / 11 = 1 block in the past
                return recipientVotes % newbalance; // we have lost 21 % 11 = 10 votes
            } else {
                // optimization for burn, vote anchor of null address does not matter
                return 0;
            }

main/contracts/Frankencoin.sol
    282: if (reserveLeft >= _amount){
            _transfer(address(reserve), msg.sender, _amount);
        } else {
            _transfer(address(reserve), msg.sender, reserveLeft);
            _mint(msg.sender, _amount - reserveLeft);
        }
        
    141: if (balance <= minReserve){
            return 0;
        } else {
            return balance - minReserve;
        }

    207: if (currentReserve < minterReserve()){
            // not enough reserves, owner has to take a loss
            return theoreticalReserve * currentReserve / minterReserve();
        } else {
            return theoreticalReserve;
        }

    282: if (reserveLeft >= _amount){
            _transfer(address(reserve), msg.sender, _amount);
        } else {
            _transfer(address(reserve), msg.sender, reserveLeft);
            _mint(msg.sender, _amount - reserveLeft);
        }
```
### RECOMMENDATION
We recommend following your own best practices as applied in the MathUtil.sol contract on line 26.
```solidity
main/contracts/MathUtil.sol
    26: cond = xOld > x ? xOld - x > THRESH_DEC18 : x - xOld > THRESH_DEC18;
```

## [R-02] | FUNCTION NAMING SUGGESTIONS
Proper use of _ as a function name prefix and a common pattern is to prefix internal and private function names with _. This pattern is correctly applied in all contracts, however there are some inconsistencies in just these contracts.

### PROOF OF CONCEPT
```solidity
// @audit Refactorize
    main/contracts/Position.sol
        169: function collateralBalance() internal view returns (uint256){
        193: function mintInternal(address target, uint256 amount, uint256 collateral_) internal {
        202: function restrictMinting(uint256 period) internal {
        232: function repayInternal(uint256 burnable) internal {
        240: function notifyRepaidInternal(uint256 amount) internal {
        268: function internalWithdrawCollateral(address target, uint256 amount) internal returns (uint256) {
        282: function checkCollateral(uint256 collateralReserve, uint256 atPrice) internal view {
        286: function emitUpdate() internal {

    main/contracts/MintingHub.sol
        287:  function returnCollateral(Challenge storage challenge, bool postpone) internal {

    main/contracts/Equity.sol
        144: function adjustTotalVotes(address from, uint256 amount, uint256 roundingLoss) internal {
        157: function adjustRecipientVoteAnchor(address to, uint256 amount) internal returns (uint256){
        172: function anchorTime() internal view returns (uint64){
        225: function canVoteFor(address delegate, address owner) internal view returns (bool) {
        266: function calculateSharesInternal(uint256 capitalBefore, uint256 investment) internal view returns (uint256) {

    main/contracts/Frankencoin.sol
        102: function allowanceInternal(address owner, address spender) internal view override returns (uint256) {

    main/contracts/ERC20.sol
        97: function allowanceInternal(address owner, address spender) internal view virtual returns (uint256) {

    main/contracts/StablecoinBridge.sol
        49: function mintInternal(address target, uint256 amount) internal {
        67: function burnInternal(address zchfHolder, address target, uint256 amount) internal {

    main/contracts/PositionFactory.sol
        37: function createClone(address target) internal returns (address result) {

    main/contracts/Ownable.sol
        39: function setOwner(address newOwner) internal {
        45: function requireOwner(address sender) internal view {
```
### RECOMMENDATION
```diff
-169: function collateralBalance() internal view returns (uint256)
+169: function _collateralBalance() internal view returns (uint256) 
```

## [R-03] AVOID BEING REDUNDANT AND STORE VALUES IN CONSTANT VARIABLES
### PROOF OF CONCEPT
```solidity
// @audit Refactorize
main/contracts/Position.sol
  if (afterFees){
            return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
        } else {
            return totalMint * (1000_000 - reserveContribution) / 1000_000;
        }
```
### RECOMMENDATION
```diff
- return totalMint * (1000_000 - reserveContribution - calculateCurrentFee()) / 1000_000;
+ uint public constant VALUE = 1000_000;
+ return totalMint * (VALUE - reserveContribution - calculateCurrentFee()) / VALUE;
```
## [S-01] WE SUGGEST USING THE OPENZEPPELIN LIBRARIES
We recommend the use of `OpenZeppelin` libraries, as you created contracts such as `IERC20`, `Ownable`, `MathUtil`, etc. `OpenZeppelin` provides robust and highly tested libraries with gas efficiency.

## [S-02] WE SUGGEST  TO USE NAMED PARAMETERS FOR MAPPING TYPE DECLARATIONS
Consider using named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability. This feature is present since Solidity 0.8.18.

`There are 7 instances where this can be refactorize.`
