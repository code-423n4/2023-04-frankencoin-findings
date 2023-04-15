[L-01] **Function restructureCapTable in Equity.sol fails to delete all addressesToWipe balances**

The function [restructureCapTable](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L303-L316) how stated in comment is an emergency function, it's useful in case the equity of Frankencoin system goes negative or even under **MINIMUM_EQUITY**, maybe some FPS holder still believe the system and they want to restore the equity at least at **MINIMUM_EQUITY**, in this case the willing of the protocol is that all the other passive FPS holder (who don't want to save Frankencoin donating to reserve) don't own any FPS after the "bailout", they want to accomplish this trough a loop that for every **addressesToWipe** delete his FPS balance trough **_burn** function in ERC20 contract but at the moment the function deletes just the first index in **addressesToWipe** without using **i** (that is also initialized at its default value wasting gas)
 ```solidity
  function restructureCapTable(address[] calldata helpers, address[] calldata addressesToWipe) public {
        require(zchf.equity() < MINIMUM_EQUITY);
        checkQualified(msg.sender, helpers);
        for (uint256 i = 0; i<addressesToWipe.length; i++){
            address current = addressesToWipe[0];
            _burn(current, balanceOf(current));
        }
```
