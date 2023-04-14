# [G-01] Use functions instead of modifiers (4 instances)
**Saved ~86 164** on Position deployment
**Saved ~40 064** on MintingHub deployment
**Saved ~88 042** on Frankencoin deployment
**Saves ~86 269** on MintingHub `openPosition`
No other noticeable gas changes were found
Example:

    function  validPos(address position) internal view{
        require(zchf.isPosition(position) == address(this), "not our pos");
    }

[Position/L366-L390](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Position.sol#L366-L390)
[MintingHub/L115](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/MintingHub.sol#L115)
[Frankencoin/L266](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L266)

# [G-02] Remove repeating functions
**Saved ~6007** on Equity deployment
**Saves ~67** on `checkQualified`
**Saves ~66** on `onTokenTransfer`
The function [`canRedeem()`](https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L127) has another version that is `canRedeem(address owner)`, since bolt of the functions are the same, `canRedeem()` just has a `return canRedeem(msg.sender)`, I suggest that `canRedeem()` be removed since it is not needed, on it's place we can just put the longer variant and input `msg.sender`.
PS. also `canRedeem()` is not used in any of the contracts. 
 
    function canRedeem() external view returns (bool){
        return canRedeem(msg.sender);
    }

