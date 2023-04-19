in the function openPosition of the MintingHub contract. The require(_initialCollateral >= _minCollateral, "must start with min col"); must be at the top of the function
This way we can optimize gas of these two lines 
```
zchf.registerPosition(address(pos));
zchf.transferFrom(msg.sender, address(zchf.reserve()), OPENING_FEE);
```

in case _minCollateral is bigger than _initialCollateral