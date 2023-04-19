# Frankencoin QA

## 1. Validate the input target in mintInternal and burnInternal if the address is not empty.

If the input target is empty, the token may become stuck. You need to check the input in the mintInternal and burnInternal in the StablecoinBridge.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L49
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L67


## 2. onTokenTransfer for CryptoFranc does not work.  

The function onTokenTransfer in StablecoinBridge does not work for CryptoFranc(0xB4272071eCAdd69d933AdcD19cA99fe80664fc08) because the the token contract does not implement the function transferAndCall.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/StablecoinBridge.sol#L76-L77

If the token contract does not have an transferAndCall implementation, the same issue may occur in the future deployment.


## 3. Validate the input _target in _mint if the address is not empty.

Mint to the zero address if the input _target is the empty address. You need to check the input.

https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L172

For example,
require(_target != address(0), “FRANKENCOIN: EMPTY ADDRESS”);
