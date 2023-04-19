## Votes calculation is `O(n^2)`, but could be improved to `O(n)`

In the `votes` function, there's the following double-loop

```solidity
for (uint i=0; i<helpers.length; i++){
    address current = helpers[i];
    /* ... */
    for (uint j=i+1; j<helpers.length; j++){
        require(current != helpers[j]); // ensure helper unique
    }
    /* ... */
}
```

Hence, if there are `n` helpers, the inner loop runs `O(n^2)` times.

This could be improved to run in `O(n)` times if the frontend sorts the `helpers` array before calling the contract, then the contract only needs
Check that each helper is larger than the previous one (a single operation, instead of `n` operations). The contract code will look like the following:

```solidity
uint160 largestHelper = 0x0;
for (uint i=0; i<helpers.length; i++){
    address current = helpers[i];
    /* ... */
    //eEnsure helper unique (assumed to be sorted by the caller)
    require(uint160(current) > largestHelper);
    largestHelper = uint160(current);
    
    /* ... */
}
```

The savings could be huge if there are a lot.

*Note:* This will revert if the helpers aren't sorted.

### Savings

I ran this function with different numbers of helpers to see the difference, and here are the results I got.

  - 1 helper --> 0.5% improvement (`25826 gas` vs `25686 gas`)
  - 10 helpers --> 50% improvement (`45450 gas` vs `30116 gas`)
  - 100 helpers --> **20,000% improvement** (20 times) (`1518368 gas` vs `73921 gas`)
  - 1000 helpers --> Without the improvement, after several minutes, Remix crashed, With the improvement, it took less than a second.

At the price of gas as of when writing this, the `100` helpers case saves around 425\$ of the cost (446\$ to 21\$).

## Use loops instead of recursion in `canVoteFor`

Recursion has problems since it takes more memory and is less efficient than regular loops. The `canVoteFor` function can be easily implemented using loops instead. The following code does so:

```solidity
// using recursion
function canVoteForRec(address delegate, address owner) public view returns (bool) {
    if (owner == delegate){
        return true;
    } else if (owner == address(0x0)){
        return false;
    } else {
        return canVoteForRec(delegate, delegates[owner]);
    }
}

// using a loop
function canVoteForLoop(address delegate, address owner) public view returns (bool) {
    address delegate_iter = delegates[owner];
    while (delegate_iter != address(0x0)) {
        if (delegate_iter == delegate) {
            return true;
        }
        delegate_iter = delegates[delegate_iter];
    }

    return false;
}
```

I would note that the gas improvement is only around 1.5% from my tests, so not that much, but still.