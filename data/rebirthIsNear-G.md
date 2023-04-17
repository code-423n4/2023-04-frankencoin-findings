File: contracts/Position.sol
===================================
Use assembly to write storage values
82: limit = _limit;
112: cooldown = expiration;
143: minted = newMinted;
165: price = newPrice;
205: cooldown = horizon;
272: cooldown = expiration;

// RECOMMENDATION
assembly {
   sstore(limit.slot, _limit)
}

assembly {
   sstore(cooldown.slot, expiration)
}

assembly {
   sstore(minted.slot, newMinted)
}

assembly {
   sstore(price.slot, newPrice)
}

assembly {
   sstore(cooldown.slot, horizon)
}

assembly {
   sstore(cooldown.slot, expiration)
}

Shift right or left instead of dividing or multiply by 2
98: uint256 reduction = (limit - minted - _minimum)/2;

// RECOMMENDATION
uint256 reduction = (limit - minted - _minimum) >> 1;
===================================

File: contracts/Equity.sol

192: for (uint i=0; i<helpers.length; i++){}

// RECOMMENDATION
uint256 len = helpers.length;

for(uint i; i<len; ++i {} 
-or
The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

for(uint i; i<len;) { 
 uchecked {++i}
}

196: for (uint j=i+1; j<helpers.length; j++){}

// RECOMMENDATION
uint256 len = helpers.length;
for(uint j=i+1; j<len; ++j) {}
-or
The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

for(uint j=i+1; j<len;) {
 uchecked {++j}
}

312: for (uint256 i = 0; i<addressesToWipe.length; i++){}

// RECOMMENDATION
uint256 len = addressesToWipe.length;
for(uint256 i; i<len; ++i) {}
-or
The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP

for(uint i; i<len;) {
 uchecked {++i}
}

