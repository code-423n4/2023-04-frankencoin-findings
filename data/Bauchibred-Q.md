# **Frankencoin QA report (Ls & NCs)**

# Low Issues

## L-01 Vulnerability to cross-chain replay attacks due to static DOMAIN_SEPARATOR

### Proof of Concept

[There is 1 instance of this issue:](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20PermitLight.sol#L61-L71)

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

### Recommendation

Check [this](https://github.com/code-423n4/2021-04-maple-findings/issues/2) issue from a prior contest for more details

## L-02 Undocumented assembly blocks

### Proof of Concept

The [PositionFactory.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L13-L20) includes a function `createClone()` with an assembly block. Even though the function is taken from EIP 1167 where the functionality is documented, assembly is a low-level language that is harder to parse by readers. Consider including extensive inline documentation clearly explaining what every single assembly instruction does. This will make it easier for users to trust the code, for reviewers to verify it, and for developers to build on top of it or update it. Note that the use of assembly discards several important safety features of Solidity, which may render the code less safe and more error-prone. Hence, consider implementing thorough tests to cover all potential use cases of the createClone function to ensure it behaves as expected.

### Recommendation

Change function `createclone()` from:

```

function createClone(address target) internal returns (address result) {
bytes20 targetBytes = bytes20(target);
assembly {
let clone := mload(0x40)
mstore(clone, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
mstore(add(clone, 0x14), targetBytes)
mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
result := create(0, clone, 0x37)
}
}

```

to:

```

function createClone(address target) internal returns (address result) {
// convert address to bytes20 for assembly use
bytes20 targetBytes = bytes20(target);
assembly {
// allocate clone memory
let clone := mload(0x40)
// store initial portion of the delegation contract code in bytes form
mstore(clone, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
// store the provided address
mstore(add(clone, 0x14), targetBytes)
// store the remaining delegation contract code
mstore(add(clone, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
// create the actual delegate contract reference and return its address
result := create(0, clone, 0x37)
}
}

```

## L-03 Use Of Block.timestamp

### Proof of Concept

There are multiple instances of this in different contracts in scope, below is one in [ERC20PermitLight.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20PermitLight.sol)

```

function permit(
address owner,
address spender,
uint256 value,
uint256 deadline,
uint8 v,
bytes32 r,
bytes32 s
) public {
require(deadline >= block.timestamp, "PERMIT_DEADLINE_EXPIRED");// @audit???

        unchecked { // unchecked to save a little gas with the nonce increment...
            address recoveredAddress = ecrecover(
                keccak256(
                    abi.encodePacked(
                        "\x19\x01",
                        DOMAIN_SEPARATOR(),
                        keccak256(
                            abi.encode(
                                // keccak256("Permit(address owner,address spender,uint256 value,uint256 nonce,uint256 deadline)"),
                                bytes32(0x6e71edae12b1b97f4d1f60370fef10105fa2faae0126114a169c64845d6126c9),
                                owner,
                                spender,
                                value,
                                nonces[owner]++,
                                deadline
                            )
                        )
                    )
                ),
                v,
                r,
                s
            );

            require(recoveredAddress != address(0) && recoveredAddress == owner, "INVALID_SIGNER");
            _approve(recoveredAddress, spender, value);
        }
    }

```

The contract developers should be aware that this does not mean current time.
Miners can influence the value of block.timestamp to perform Maximal Extractable Value (MEV) attacks. The use of now creates a risk that time manipulation can be performed to manipulate price oracles. Miners can modify the timestamp by up to 900 seconds.

### Recommendation

Use block.number instead of block.timestamp to reduce the risk of MEV attacks. Check if the timescale of the project occurs across years, days and months rather than seconds. If possible, it is recommended to use Oracles.

## L-04 MintingHub.minBid() would not always work as designed to

### Proof of Concept

Check [MintingHub.minBid():](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L185-L190)

```
    /**
     * The minimum bid size for the next bid. It must be 0.5% higher than the previous bid.
     */
    function minBid(Challenge storage challenge) internal view returns (uint256) {
        return (challenge.bid * 1005) / 1000;
    }
```

From code block we can see that in a few cases this function would not return the right `minBid`, this only works correctly if previous bid, i.e challenge.bid >= 200, anything less than this and the function returns previous `challenge.bid` after computation instead of acceptable `minBid` due to the rounding down

### Recommendation

Include this in the documentation to make users know about this before hand so as not to get confused

## L-05 Stricter than needed inequalities may affect borderline scenarios

### Proof of Concept

[calculateProceeds():](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L284-L297)

```
    /**
     * @notice Calculate ZCHF received when depositing shares
     * @param shares number of shares we want to exchange for ZCHF,
     *               in dec18 format
     * @return amount of ZCHF received for the shares
     */
    function calculateProceeds(uint256 shares) public view returns (uint256) {
        uint256 totalShares = totalSupply();
        uint256 capital = zchf.equity();
        require(shares + ONE_DEC18 < totalShares, "too many shares"); // make sure there is always at least one share
        uint256 newTotalShares = totalShares - shares;
        uint256 newCapital = _mulD18(capital, _power3(_divD18(newTotalShares, totalShares)));
        return capital - newCapital;
    }
```

L293:
`        require(shares + ONE_DEC18 < totalShares, "too many shares"); // make sure there is always at least one share`
in the case where there is only one share the above requirement fails.

### Recommendation

Reconsider strict inequalities and relax them if possible.
Change L293 from:
`        require(shares + ONE_DEC18 < totalShares, "too many shares"); // make sure there is always at least one share`

to:

`        require(shares + ONE_DEC18 <= totalShares, "too many shares"); // make sure there is always at least one share`

## L-06 `PositionFactory.createNewPosition()` missing essential onlyHub() modifier

### Proof of Concept

the [PositionFactory.createNewPosition()](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L13-L20)

```

    /**
     * Create a completely new position in a newly deployed contract.
     * Must be called through minting hub to be recognized as valid position.
     */
    function createNewPosition(address _owner, address _zchf, address _collateral, uint256 _minCollateral,
        uint256 _initialLimit, uint256 initPeriod, uint256 _duration, uint256 _challengePeriod,
        uint32 _mintingFeePPM, uint256 _liqPrice, uint32 _reserve)
        external returns (address) {
        return address(new Position(_owner, msg.sender, _zchf, _collateral,
            _minCollateral, _initialLimit, initPeriod, _duration,
            _challengePeriod, _mintingFeePPM, _liqPrice, _reserve));
    }

```

from documentation we can see that the function must be called through the minting hub to be recognised as a valid position, but no access control whatsoever is applied to this function and being an external function anyone could successfully call it

### Recommendation

Advised to introduce an onlyHub modifier as is being used in [Position.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol)

```

'''
modifier onlyHub() {
if (msg.sender != address(hub)) revert NotHub();
\_;
}

```

## L-07 `PositionFactory.clonePosition()` is missing essential check that position exists

### Proof of Concept

The[function clonePosition()](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/PositionFactory.sol#L22-L34) clones an existing position, which could also be a clone of another clone, though this function executes properly when the address of the position that's going to be cloned exists, in the case the address doesn't exist the function works in an unfathomable manner, and end users might not know why the transaction isn't going through

### Recommendation

A check should be added to make sure that the position that's to be cloned exists before executing remaining code block

## L-08 Floating Pragma Solidity Version

### Proof of Concept

Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

https://swcregistry.io/docs/SWC-103

### Recommendation

Lock the pragma version to the same version as used in the other contracts and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.

## L-09 Lack of complete checks in the modified `ERC20.approve()`

### Proof of Concept

[`ERC20.approve()](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L208-L224)

```

/\*\*
_ @dev Sets `amount` as the allowance of `spender` over the `owner`'s tokens.
_
_ This is internal function is equivalent to `approve`, and can be used to
_ e.g. set automatic allowances for certain subsystems, etc. \*
_ Emits an `Approval` event.
_
_ Requirements:
_
_ - `owner` cannot be the zero address.
_ - `spender` cannot be the zero address.
\*/
function \_approve(address owner, address spender, uint256 value) internal {
\_allowances[owner][spender] = value;
emit Approval(owner, spender, value);
}

```

As stated in the Natspec of this function both `owner` and `sender` cannot be the zero address, however no checks whatsoever are added to the function to ensure that the execution errors if either of `owner` or `sender` is the zero address

### Recommendation

Add the following lines to the function block before updating the allowance

```

require(owner != address(0), "ERC20: approve from the zero address");
require(spender != address(0), "ERC20: approve to the zero address");

```

## L-10 Immutable addresses lack zero-address check

### Proof of Concept

[Equity.sol#constructor](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L94)

[Position.sol#constructor](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L50-L70)

[StablecoinBridge.sol#constructor](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L26-L31)

Multiple instances in codes, which include:

Constructors should check the address written in an immutable address variable is not the zero address when the contract is being deployed by the team.

### Recommendation

Add a zero address check for the immutable variables/addresses aforementioned.

# Non-critical Issues

## NC-01 No error messages in critical functions like `ERC20._transfer()`

### Proof of Concept

[ERC20.\_transfer():](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L151-L159)

```

    function _transfer(address sender, address recipient, uint256 amount) internal virtual {
        require(recipient != address(0));

        _beforeTokenTransfer(sender, recipient, amount);
        if (_balances[sender] < amount) revert ERC20InsufficientBalance(sender, _balances[sender], amount);
        _balances[sender] -= amount;
        _balances[recipient] += amount;
        emit Transfer(sender, recipient, amount);
    }

```

### Recommendation

Though it's been documented that the [ERC20.sol](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol#) contract is modified/adjusted to remove unnecessary require statements/messages to save gas, the error messages attached to functions like transfer are very important and should remain

## NC-02 Zero checks in StablecoinBridge.onTokenTransfer()

### Proof of Concept

```

    function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool){
        if (msg.sender == address(chf)){
            mintInternal(from, amount);
        } else if (msg.sender == address(zchf)){
            burnInternal(address(this), from, amount);
        } else {
            require(false, "unsupported token");
        }
        return true;
    }

```

### Recommendation

A check should be added that `amount > 0`

## NC-03 More readable constants

### Proof of Concept

`Frankencoin.minterReserve()`:

```
   function minterReserve() public view returns (uint256) {
      return minterReserveE6 / 1000000;
   }
```

Some constant values are difficult to read in one time because they have at lot of 0's. Solidity allows \_ to separate series of zeroes.

### Recommendation

The following should be replaced:

1000000 with 1_000_000

## NC-04 Fix typos

### Proof of Concept

[Frankencoin.allowanceInternal:](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L96-L111)

L100: `   * giving them arbitraty allowances does not pose an additional risk.`

### Recommendation

Change from:

`   * giving them arbitraty allowances does not pose an additional risk.`

to

`   * giving them arbitra`R`y allowances does not pose an additional risk.`

## NC-05 Lack of error message in important instance in the `MintingHub.splitChallenge()` function

### Proof of Concept

[MintingHub.splitChallenge()](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/MintingHub.sol#L150-L179)

```
    /**
     * Splits a challenge into two smaller challenges.
     * This can be useful to guard an attack, where a challenger launches a challenge so big that most bidders do not
     * have the liquidity available to bid a sufficient amount. With this function, the can split of smaller slices of
     * the challenge and avert it piece by piece.
     */
    function splitChallenge(uint256 _challengeNumber, uint256 splitOffAmount) external returns (uint256) {
        Challenge storage challenge = challenges[_challengeNumber];
        require(challenge.challenger != address(0x0));
        Challenge memory copy = Challenge(
            challenge.challenger,
            challenge.position,
            splitOffAmount,
            challenge.end,
            challenge.bidder,
            (challenge.bid * splitOffAmount) / challenge.size
        );
        challenge.bid -= copy.bid;
        challenge.size -= copy.size;


        uint256 min = IPosition(challenge.position).minimumCollateral();
        require(challenge.size >= min);
        require(copy.size >= min);


        uint256 pos = challenges.length;
        challenges.push(copy);
        emit ChallengeStarted(challenge.challenger, address(challenge.position), challenge.size, _challengeNumber);
        emit ChallengeStarted(copy.challenger, address(copy.position), copy.size, pos);
        return pos;
    }
```

in L171-172:
`        require(challenge.size >= min);
        require(copy.size >= min);`

Any of these two cases `challenge.size < min || copy.size < min` would cause the execution to revert, but end users might not know that this is why their transaction is reverting

### Recommendation

Add error messages in essential instances of code

## NC-06 Change 52 weeks to 365 days

### Proof of Concept

[There is one instance of this](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L29)

```

horizon = block.timestamp + 52 weeks;

```

### Recommendation

Advised it's changed to + 365 days instead of 52 weeks as 52 weeks is a day less than 365 days, if the intention is to wait for a year after block.timestamp.
