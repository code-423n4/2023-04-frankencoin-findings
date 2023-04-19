# [L-01] Rebasing token is not supported

As mentioned in the previous audit of Frankencoin conducted by Blockbite, certain types of tokens (those with transfer fees) should not be used as collateral in the system. Additionally, there is another kind of token that has not been mentioned: rebasing tokens.

[https://docs.frankencoin.com/governance/acceptable-collateral](https://docs.frankencoin.com/governance/acceptable-collateral)

In the documentation, I did not find any information about rebasing tokens. Rebasing tokens, like AMPL, are not supported by the system. This is because, during the challenging process, if the token's amount changes, it becomes difficult to consistently challenge and liquidate the position. The complexity increases even further if the rebasing token's amount changes during the challenging process, making the process uncertain.

## Recommendation

Include a notice stating "avoid using rebasing tokens in the system" in the ***[Acceptable Collateral](https://docs.frankencoin.com/governance/acceptable-collateral)***
 chapter of the documentation. 

# [L-02] The risk of share holders being absent

Currently the design of the system relies heavily on the shareholders for vetoing the position or minters to guard the system. 

As mentioned in the comments of the code base: 

```solidity
File: Frankencoin.sol
09: /**
10:  * The Frankencoin (ZCHF) is an ERC-20 token that is designed to track the value of the Swiss franc.
11:  * It is not upgradable, but open to arbitrary minting plugins. These are automatically accepted if none of the
12:  * qualified pool share holders casts a veto, leading to a flexible but conservative governance.
13:  *
14:  * The underlying assumption is that there is one or more qualified pool share (FPS) holders that watch the proposals
15:  * and veto if necessary. At the same time, it is also assumed that no one vetoes all of them, thereby starving the
16:  * system. The system can only function as long as all the qualified shareholders act in the interest of the system
17:  * or at least do not actively sabotage it. As long as everyone believes that the governance works well, no one will
18:  * ever make an unsound proposal and it won't be necessary to ever cast a veto, making the system self-governing.
```

The presence of shareholders is there and acting in the interest of the system or themselves. The design of vetoing the proposals by anyone also must have some shareholder to veto the minter or deny the positions. 

By any chance, if the shareholder is not there to perform, the system can collapse anytime when malicious minter or position is created. 

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Frankencoin.sol#L152-L157](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Frankencoin.sol#L152-L157)

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Position.sol#L108-L114](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Position.sol#L108-L114)

## Recommendation

Choose one of the following:

- Allow shareholders who actively participate in the process of denying minters or positions to receive a small fraction of the fee paid by the proposer. This will increase the participation incentive for shareholders.
- Alternatively, consider allowing only eligible shareholders to approve minters and positions. If a certain threshold is reached, the minter or position will become active.
- Add a function to revoke the minter even after they have been validated. This could be triggered by governance.

The sponsor may choose to maintain the current system. If so, always be aware that any absence of shareholder participation could lead to the collapse of the system since its inception.

# [L-03] Deal with extreme market condition

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/MintingHub.sol#L110-L111](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/MintingHub.sol#L110-L111)

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Frankencoin.sol#L87](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Frankencoin.sol#L87)

In the protocol, it takes the transaction sender’s fee to propose a new minter or position. 

This works as a defense to spamming. And it is a good measure to be taken like this in general. But in some extreme conditions, things may change. It is easy to assume when the protocol is under the water, which means when the price of `zchf` token de-pegs and  becomes cheap, this kind of measure taken to avoid spamming may not work. 

The cost to spam the protocol can decrease, and at the end of the day, the system can be flooded with spamming transactions. So, it is better to let the protocol have the ability to change the token of the fee by governance like the veto system. In this case, the protocol can take certain actions to prevent the spamming attacks by changing the proposing fee token into something else ,say USDC, even when the market is in extreme condition. 

(It may not happen, but it is better to be prepared for the worst case scenario.) 

## Recommendation

Create a way to change the token of fee by governance. 

# [N-01] No error message

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Equity.sol#L194](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Equity.sol#L194)

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/MintingHub.sol#L171-L172](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/MintingHub.sol#L171-L172)

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/MintingHub.sol#L158-L172](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/MintingHub.sol#L158-L172)

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/MintingHub.sol#L254](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/MintingHub.sol#L254)

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Position.sol#L53](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Position.sol#L53)

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Equity.sol#L310](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Equity.sol#L310)

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Equity.sol#L194-L198](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Equity.sol#L194-L198)

Using a custom error pattern is most recommended because it saves gas. If you want to maintain the `require` patter, adding an error message to the `require` is supposed to be the best practice. Following it is suggested. 

# [N-02] Missing parameter in document

There is a missing `_initPeriodSeconds` parameter in the document of code base. 

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/MintingHub.sol#L76-L87](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/MintingHub.sol#L76-L87)

```solidity
File: MintingHub.sol
76:      * @param _collateralAddress        address of collateral token
77:      * @param _minCollateral     minimum collateral required to prevent dust amounts
78:      * @param _initialCollateral amount of initial collateral to be deposited
79:      * @param _mintingMaximum    maximal amount of ZCHF that can be minted by the position owner // @audit lacks the `_initPeriodSeconds` param in the doc
80:      * @param _expirationSeconds position tenor in unit of timestamp (seconds) from 'now'
81:      * @param _challengeSeconds  challenge period. Longer for less liquid collateral.
82:      * @param _mintingFeePPM     ppm of minted amount that is paid as fee to the equity contract
83:      * @param _liqPrice          Liquidation price with (36 - token decimals) decimals,
84:      *                           e.g. 18 decimals for an 18 decimal collateral, 36 decimals for a 0 decimal collateral.
85:      * @param _reservePPM        ppm of minted amount that is locked as borrower's reserve, e.g. 20%
86:      * @return address           address of created position
87:      */
```

# [N-03] Change to a more proper name

Change the function name from `validPos` to `validPosition`. `validPosition` is easier to understand. 

```
File: MintingHub.sol
114: 
115:     modifier validPos(address position) { // @audit `validPosition` is eaier to understand
116:         require(zchf.isPosition(position) == address(this), "not our pos");
117:         _;
118:     }
119:
```

# [N-04] Should check if the argument is zero address

Should check if the argument is zero address. 

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Ownable.sol#L35-L43](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Ownable.sol#L35-L43)

Add  this line into the code base: `if (newOwner == address(0)) revert ZeroAddressNotAllowed();` .

# [N-05] Immutable variables should be capitalized

Immutable variables should be capitalized to follow the best practice. 

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Position.sol#L28-L39](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Position.sol#L28-L39)

```solidity
File: Position.sol
28: 
29:     uint256 public immutable start; // timestamp when minting can start
30:     uint256 public immutable expiration; // timestamp at which the position expires     // @audit immutable variable name should be capitalized.
31: 
32:     address public immutable original; // originals point to themselves, clone to their origin
33:     address public immutable hub; // the hub this position was created by
34:     IFrankencoin public immutable zchf; // currency
35:     IERC20 public override immutable collateral; // collateral
36:     uint256 public override immutable minimumCollateral; // prevent dust amounts
37: 
38:     uint32 public immutable mintingFeePPM;
39:     uint32 public immutable reserveContribution; // in ppm
```

## Recommendation

```diff
-		uint256 public immutable start; // timestamp when minting can start
+		uint256 public immutable START; // timestamp when minting can start
-    uint256 public immutable expiration; // timestamp at which the position expires     // @audit immutable variable name should be capitalized.
+    uint256 public immutable EXPIRATION; // timestamp at which the position expires    

-    address public immutable original; // originals point to themselves, clone to their origin
+    address public immutable ORIGINAL; // originals point to themselves, clone to their origin
-    address public immutable hub; // the hub this position was created by
+    address public immutable HUB; // the hub this position was created by
-    IFrankencoin public immutable zchf; // currency
+    IFrankencoin public immutable ZCHF; // currency
-    IERC20 public override immutable collateral; // collateral
+    IERC20 public override immutable COLLATERAL; // collateral
-    uint256 public override immutable minimumCollateral; // prevent dust amounts
+    uint256 public override immutable MINNIMUM_COLLATERAL; // prevent dust amounts

-    uint32 public immutable mintingFeePPM;
+    uint32 public immutable MINTINT_FEE_PPM;
-    uint32 public immutable reserveContribution; // in ppm
+    uint32 public immutable RESERVE_CONTRIBUTION; // in ppm
```

# [N-06] Error should be put in the beginning of the file

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Position.sol#L103-L104](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Position.sol#L103-L104)

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Frankencoin.sol#L91-L94](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Frankencoin.sol#L91-L94)

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Equity.sol#L214](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Equity.sol#L214)

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/MintingHub.sol#L231-L233](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/MintingHub.sol#L231-L233)

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Ownable.sol#L25](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Ownable.sol#L25)

The error should not be put in the middle of the code in the contract. It should be put outside the contract and in the beginning of the file. 

This is an example of where the error should be put: 

```solidity
error InvalidAmount (uint256 sent, uint256 minRequired); // should be put here

contract TestToken {
    mapping(address => uint) balances;
    uint minRequired;

    constructor (uint256 _minRequired) {
        minRequired = _minRequired;
    }

    function list() public payable {
        uint256 amount = msg.value;
        if (amount < minRequired) {
            revert InvalidAmount({
                sent: amount,
                minRequired: minRequired
            });
        }
        balances[msg.sender] += amount;
    }
}
```

# [N-07] Add contract name into the error message

I noticed in the code base, the error messages in a lot of places are not following the best practice by adding the contract name into it. 

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Equity.sol#L193-L198](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Equity.sol#L193-L198)

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/StablecoinBridge.sol#L49-L51](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/StablecoinBridge.sol#L49-L51)

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Equity.sol#L241-L256](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/Equity.sol#L241-L256)

## Recommendation

I recommend use the custom error pattern to save gas. But if the sponsor wants to maintain using the current error pattern, then add the contract name in front of the error message to make it clearer to understand. 

This is an example: 

```solidity
	contract Contractname {
    function add(uint256 a, uint256 b) internal pure returns (uint256 c) {
        require((c = a + b) >= b, "Contractname: Add Overflow"); // here add the contract name
    }
```

# [N-08] Incorrect explanation in documentation

There is a wrong explanation in the code base. 

- It is not "the same number of source coin". Instead, it should be "the same amount of source coin".
- And the token should be sent to the address of `target` , not the address of `caller`.

```solidity
File: StablecoinBridge.sol
59:     /**
60:      * Burn the indicated amount of Frankencoin and send the same number of source coin to the caller. 
61:      * No allowance required.
62:      */
63:     function burn(address target, uint256 amount) external {
64:         burnInternal(msg.sender, target, amount);// @audit non-critical incorrect doc explaination. it is not "the same number of source coin". it is "the same amount of source coin". And the token should be sent to the "target" address, not the "caller".
65:     }
66: 
67:     function burnInternal(address zchfHolder, address target, uint256 amount) internal {
68:         zchf.burn(zchfHolder, amount); // @audit-info burned the holder's token of zchf, not the caller's token of zchf.
69:         chf.transfer(target, amount);
70:     }
71:
```

# [N-09] Adding a function to show reserve condition of the StablecoinBridge

according to documentation in the code base: 

> “System participants should closely watch the amount of other stable coins flowing in and out. Having a lot of outflow could be an indication that it is too cheap to mint Frankencoins, i.e. implied interest rates being too low. Having large inflows could be an indication that going into Frankencoins is too attractive and interest rates too high."
> 

I think it is better to add a function to check the amount of other stablecoin flowing in and out.

```solidity
function checkCondition() external view returns (uint256) {
   return chf.balanceOf(address(this)) * ONE_DEC18 / limit; 
}
```

in this way we can check condition of how much the amount of stable coin flowing in and out in an easier way. And it shows the healthiness of the protocol. 

# [N-10] Changing the contract name to make more sense

[https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/StablecoinBridge.sol#L11](https://github.com/code-423n4/2023-04-frankencoin/blob/fc86a4ada6dfaae3990609e3cc531f5c85a8f785/contracts/StablecoinBridge.sol#L11)

The contract to swap ZCHF to the other stable coin is called `StablecoinBridge`. Because a Bridge in the blockchain industry can mean a totally different thing, I suggest rename it to `StablecoinSwapper` or `StablecoinExchange` so that they make more sense.