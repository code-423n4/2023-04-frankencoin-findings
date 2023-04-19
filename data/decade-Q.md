## 1. Meta Transactions not supported due to removal of Context. 
- Removing GSN Context will make protocol unsupportive of meta transactions. 
- Meta Transactions require _msgSender() and _msgData() to proceed. 
- This will not be fulfilled, and certain products build on top of protocols like Biconomy won't be supports. 
- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L7
- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Ownable.sol#L6

---
## 2. Add a MAX application_fee check in `suggestMinter()` 
- _applicationfee is directly transferred to the reserve. Application fee should have maximim application_fee allowed check to avoid unwanted transfer to reserve or any frontend miscalculations. 
- Also _applicationFee should be renamed to priority fee is a any suggestor is willing to importance towards her application. 
- https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L83-L90 
```
      _transfer(msg.sender, address(reserve), _applicationFee);
```
## 3. Check for `_minApplicationPeriod` to be greater than zero in constructor.
- _minApplicationPeriod is an important variable that restricts adding of new minter for a stipulated period. 
- It is recommeded to check _minApplicationPeriod set to be greater than zero or some hardcoded values, to avoid. 
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L59-L62
```
  constructor(uint256 _minApplicationPeriod) ERC20(18){
      MIN_APPLICATION_PERIOD = _minApplicationPeriod;
      reserve = new Equity(this);
   }
```
Source : https://github.com/Uniswap/v3-core/blob/main/audits/tob/audit.pdf

## 4. No ability to update `reserve` in Frankencoin.sol in an unlikely scenario. 
- Although protocol design is not dependent on owner type scenario. It is worth to note, that in an unlikely event of attack. There should be functionality to update reserve in frankencoin by the admin. 
https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Frankencoin.sol#L31

## 5. Consider Two-step ownership transfer for all Position contracts. 
- Two-step ownership transfer prevents of falling of ownership of Position contract to unintended address which can be either malicious, inactive or contract address that are unable to any transaction. 
- Currently, `Ownable` contracts directly transfers ownership of the contract in a single transaction. 
- https://github.com/solodit/solodit_discussion/discussions/8403
