## [G-1] shifting state variables can save gas
Generally we know that 1 storage slot can hold 32 bytes 
so here we trying to pack the state variables to save storage slot
There are 2 instances of it

1st instance is at [Position.sol#L38-L39](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L38-L39)
here we are taking `uint32 public immutable mintingFeePPM;`(4 bytes) and `uint32 public immutable reserveContribution;` (4 bytes) and packing with `address public immutable original;` (20 bytes)
so we are clubing them to save 1 storage slot which can save ~ 2100 of gas

2nd instance is at [Equity.sol#L39-L61](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L39-L61) 

    uint32 public constant VALUATION_FACTOR = 3; // 4 bytes 

    uint256 private constant MINIMUM_EQUITY = 1000 * ONE_DEC18; // 32 bytes

    
    uint32 private constant QUORUM = 300; // 4 bytes

    
    uint8 private constant BLOCK_TIME_RESOLUTION_BITS = 24; // 1 byte

    uint256 public constant MIN_HOLDING_DURATION = 90*7200 << BLOCK_TIME_RESOLUTION_BITS;  // 32 bytes

    Frankencoin immutable public zchf; // 20 bytes

so here 5 storage slots have been used and we can bring it to 3 storage slots

    uint8 private constant BLOCK_TIME_RESOLUTION_BITS = 24; // 1 byte

    uint32 public constant VALUATION_FACTOR = 3; // 4 bytes 

    uint32 private constant QUORUM = 300; // 4 bytes

    Frankencoin immutable public zchf; // 20 bytes

    uint256 private constant MINIMUM_EQUITY = 1000 * ONE_DEC18; // 32 bytes

    uint256 public constant MIN_HOLDING_DURATION = 90*7200 << BLOCK_TIME_RESOLUTION_BITS;  // 32 bytes


now only 3 storage slots have been used and we saved 2 storage slots which saves ~ 4200 of gas