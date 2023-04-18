Possible Out of Gas failure due to infinite loop 
---

Instances include:

https://github.com/code-423n4/2023-04-frankencoin/blob/f86279e76fd9f810d2a25243012e1be4191a547e/contracts/Equity.sol#L225-L233

The recursive function calls itself, if the array of helpers has a duplicate, the function will continue to call itself until the transaction runs out of gas. Worst case, if no gas limit is specified, the user will run out of gas when calling this function. 


```
  function canVoteFor(address delegate, address owner) internal view returns (bool) {
        if (owner == delegate){ //@audit might want to check who the owner is in this case. Most likely the owner of the contract???
            return true;
        } else if (owner == address(0x0)){
            return false;
        } else {
            return canVoteFor(delegate, delegates[owner]); //@audit this function calls itself ? 
        }
    }

```

Recommendation:


Include a condition to ensure the previous address being targeted is not equal to the current address.


```


unction canVoteFor(address delegate, address owner) internal view returns (bool) {
    if (owner == delegate) {
        return true;
    } else if (owner == address(0x0)) {
        return false;
    } else {
        // Keep track of visited addresses to prevent infinite loops
        mapping (address => bool) visited;
        visited[owner] = true;
        address currentOwner = delegates[owner];
        while (currentOwner != delegate && currentOwner != address(0x0) && !visited[currentOwner]) {
            visited[currentOwner] = true;
            currentOwner = delegates[currentOwner];
        }
        return currentOwner == delegate;
    }
}


```