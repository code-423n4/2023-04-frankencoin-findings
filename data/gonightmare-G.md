https://github.com/code-423n4/2023-04-ens/blob/45ea10bacb2a398e14d711fe28d1738271cd7640/contracts/resolvers/profiles/ABIResolver.sol#L53         

/////////        

https://github.com/code-423n4/2023-04-ens/blob/45ea10bacb2a398e14d711fe28d1738271cd7640/contracts/resolvers/profiles/DNSResolver.sol#L95   

//////////    

https://github.com/code-423n4/2023-04-ens/blob/45ea10bacb2a398e14d711fe28d1738271cd7640/contracts/utils/UniversalResolver.sol#L283 //////////////// 




IMPACT 

Gas Optimizations


PROOF OF CONCEPT 

When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0. 

TOOLS USED 
N/A

RECCOMENDED MITIGATION STEPS 

2023-04-ens/contracts/resolvers/profiles/ABIResolver.sol::53 => abiset[contentType].length != 0


2023-04-ens/contracts/resolvers/profiles/DNSResolver.sol::95 => if (name.length != 0) {


2023-04-ens/contracts/utils/UniversalResolver.sol::283 => if (metaData.length != 0) {








