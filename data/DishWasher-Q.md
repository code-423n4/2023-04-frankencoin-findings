## QA report

1.) no contract is implementing IERC677.
    - Equity.sol and StablecoinBridge both implement onTokenTransfer but dont inherit the method from the ERC677 interface
    - fix: inherit from IERC677 and add override keyword to methods


2.) allowanceInternal function in ERC20.sol seems unnecessary given allowance function.


3.) the way in which the current implementation is keeping track of votes looses the distinction between different owned shares (see adjustRecipientVotesAnchor)
    - if I own some shares for 1000 blocks and others for 100 blocks
        I would like to sell or move those that I own for a shorter amount of time. At the moment, whenever new shares are bought, the holding time is averaged, making this desired behavior not possible.


4.) In general, it might be a good idea to create a Minter interface that every proposed minter has to implement. This would create more clarity in the codebase and also more control over the specific implementation of minters.


5.) in MathUtils.sol cubicRoot function power3 function is not used although it is implemented


6.) add description of _initPeriodSeconds parameter in openPosition function MintingHub


7.) minBid() in the MintingHub.sol contract is supposed to be the minimal size for a next bid. 
However, if that next bid is large enough to avert the challenge it is accepted even though it might be smaller than minBid()
