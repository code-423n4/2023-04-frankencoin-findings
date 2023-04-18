[L-01] **Function restructureCapTable in Equity.sol fails to delete all addressesToWipe balances**

The function [restructureCapTable](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L303-L316) in Equity.sol how stated in comment is an emergency function, it's useful in case the equity of Frankencoin system goes negative or even under **MINIMUM_EQUITY**, maybe some FPS holder still believe the system and they want to restore the equity at least at **MINIMUM_EQUITY**, in this case the willing of the protocol is that all the other passive FPS holder (who don't want to save Frankencoin donating to reserve) don't own any FPS after the "bailout", they want to accomplish this trough a loop that for every **addressesToWipe** delete his FPS balance trough **_burn** function in ERC20 contract but at the moment the function deletes just the first index in **addressesToWipe** without using **i** (that is also initialized at its default value wasting some gas)
 ```solidity
  function restructureCapTable(address[] calldata helpers, address[] calldata addressesToWipe) public {
        require(zchf.equity() < MINIMUM_EQUITY);
        checkQualified(msg.sender, helpers);
        for (uint256 i = 0; i<addressesToWipe.length; i++){
            address current = addressesToWipe[0];
            _burn(current, balanceOf(current));
        }
```
[I-01]**Some comments from openzeppelin/ERC20 are still there but are useless**
In [ERC20](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/ERC20.sol) there is a [comment](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L36-L39) useless stating the difference from openzeppelin/ERC20  and the acknowledgement of previus audit specifically I-2 insecure approval implementation on ERC20.

Also in [_burn function](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L195-L199) , and [approve](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L102-L107), [_transfer](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L147), [_approve](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L216-L220), [transfer](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L80-L84) stating the willing of the protocol [to remove unnecessary require statement](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/ERC20.sol#L6) that is not advised

[I-02]**Some comments has wrong description**
In [StablecoinBridge](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol)
there is [comment](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L59-L62) that explain burn function but it actually describes the other burn [function](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L55-L57) it is recommended to move the actual comment to the other function and writes new comments for this burn function
```solidity
    /**
     * Burn the indicated amount of Frankencoin and send the same number of source coin to the caller.
     * No allowance required.
     */
    function burn(uint256 amount) external {
        burnInternal(msg.sender, msg.sender, amount);
    }

    /**
     * Burn the indicated amount of Frankencoin and send the same number of source coin to the target specified by the caller.
     * No allowance required.
     */
    function burn(address target, uint256 amount) external {
        burnInternal(msg.sender, target, amount);
    }
```
In the same contract in [mint function](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L40-L47) there is a comment that can be dangerous 
```solidity
 /**
     * Mint the target amount of Frankencoins, taking the equal amount of source coins from the sender.
     * This only works if an allowance for the source coins has been set and the caller has enough of them.
     */
    function mint(address target, uint256 amount) public {
        chf.transferFrom(msg.sender, address(this), amount);
        mintInternal(target, amount);
    }
```
is recommended changing it to:

```solidity
 /**
     * Mint to the target specified by the caller the specified amount of Frankencoins, taking the equal amount of source coins from the sender.
     * This only works if an allowance for the source coins has been set and the caller has enough of them.
     */
    function mint(address target, uint256 amount) public {
        chf.transferFrom(msg.sender, address(this), amount);
        mintInternal(target, amount);
    }
```
stating that the target is who will receive the zchf in [**mintInternal**](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/StablecoinBridge.sol#L49-L53)



