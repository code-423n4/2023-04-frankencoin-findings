### This DOMAIN_SEPARATOR() is not standard implementation
As the code below, the domain separator is built with `chainId` and `verifyingContract`. 

```solidity
    function DOMAIN_SEPARATOR() public view returns (bytes32) {
        return
            keccak256(
                abi.encode(
                    //keccak256("EIP712Domain(uint256 chainId,address verifyingContract)");
                    bytes32(0x47e79534a245952e8b16893a336b85a3d9ea9fa8c573f3d803afb92a79469218),
                    block.chainid,
                    address(this)
                )
            );
    }
```
However, the standard implementation looks like this: 
keccak256("EIP712Domain(string name,string version,uint256 chainId,address verifyingContract,bytes32 sal)");

```solidity
name: the dApp or protocol name (e.g. "Uniswap")
version: version number of your dApp or platform
chainId: EIP-155 chain id
verifyingContract: The Ethereum address of the contract that will verify the signature (accessible via this)
salt: A unique 32-byte value hardcoded into both the contract and the dApp meant as a last-resort to distinguish the dApp from others

```