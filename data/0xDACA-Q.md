
## A wrong index is used in `restructureCapTable`

In the function `restructureCapTable` there's the following loop

```solidity
for (uint256 i = 0; i<addressesToWipe.length; i++) {
    address current = addressesToWipe[0];
    _burn(current, balanceOf(current));
}
```

On each iteration of the loop, `current = addressesToWipe[0]` is the same thing, so only the first iteration of the loop does something, and the next iterations do nothing.
The code should be `current = addressesToWipe[i]` instead (notice the `[i]` instead of `[0]`).

*Note:* This was not caught in the tests because in the tests this function is called with a list with a single element.