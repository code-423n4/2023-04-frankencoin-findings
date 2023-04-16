# **Equity.sol**

## 1.Improvement of function *votes(address, address[])*
This function exhibits several areas for improvement:

-Optimization of *helpers.length*: In the for loops, *helpers.length* is repeatedly called. To optimize gas usage, consider storing the length in a local variable and referencing the variable within the loop instead of recalculating the length each time.

    uint256 helpersLength = helpers.length;
    for (uint256 i = 0; i < helpersLength; i++) {
      ...
    }

-Length Limitation: To mitigate potential Denial of Service (DoS) attacks, impose a limit on the helpers array length. This ensures that the function execution time remains within reasonable bounds.

    require(helpers.length <= maxLength, "helpers array exceeds allowed length");

-Early require Statements: The current implementation places require statements near the end of the function. This can lead to wasted gas fees for users if the function reverts after executing most of the loops. To address this, move the require statements to the beginning of the function, ensuring that any conditions are checked before executing loops.

-Nested Loop Optimization: The function contains nested loops, which can be highly inefficient and lead to excessive gas fees. To enhance efficiency, consider using a mapping instead of nested loops.

Taking all of these issues into account, I propose to change the function as follows:

    function votes(address sender, address[] calldata helpers) public view returns (uint256) {
      uint256 maxLength = 100; // Set an appropriate limit
      require(helpers.length <= maxLength, "Array size exceeds the allowed limit");

      uint256 helperCount = helpers.length;

      // Validate input and ensure helpers' uniqueness
      mapping(address => bool) memory helperSet;
      for (uint i = 0; i < helperCount; i++) {
        address current = helpers[i];
        require(current != sender, "Helper cannot be the same as sender");
        require(canVoteFor(sender, current), "Sender cannot vote for this helper");
        require(!helperSet[current], "Helper must be unique");
        helperSet[current] = true;
      }

      // Calculate votes
      uint256 _votes = votes(sender);
      for (uint i = 0; i < helperCount; i++) {
        _votes += votes(helpers[i]);
      }

      return _votes;
    }

By implementing these improvements, the function can be optimized for gas efficiency, security, and readability.

## 2.Inefficient placement of require statement in the *onTokenTransfer* function

The require statement checking whether the totalSupply() is less than 2**128 is currently located at the end of the function. This placement could lead to unnecessary gas fees for users in case the function reverts at this point.

To optimize gas usage and prevent unnecessary gas fees, it is recommended to move the require statement before the _mint call. Additionally, update the condition to check whether the totalSupply() plus the new shares is less than 2**128. The modified function should look like this:

    function onTokenTransfer(address from, uint256 amount, bytes calldata) external returns (bool) {
      require(msg.sender == address(zchf), "caller must be zchf");
      uint256 equity = zchf.equity();
      require(equity >= MINIMUM_EQUITY, "insuf equity"); // ensures that the initial deposit is at least 1000 ZCHF

      // Assign 1000 FPS for the initial deposit, calculate the amount otherwise
      uint256 shares = equity <= amount ? 1000 * ONE_DEC18 : calculateSharesInternal(equity - amount, amount);

      // limit the total supply to a reasonable amount to guard against overflows with price and vote calculations
      // the 128 bits are 68 bits for magnitude and 60 bits for precision, as calculated in an above comment
      require(totalSupply() + shares < 2**128, "total supply exceeded");

      _mint(from, shares);
      emit Trade(msg.sender, int(shares), amount, price());

      return true;
    }

# **Frankencoin.sol**

## 1.block.timestamp 

Instead of using *block.timestamp*, use *block.number* as it cannot be manipulated by miners. However, this requires changes to the code structure. Instead of relying on durations, you will need to use block numbers. For instance, if you want to permit a user to mint after 7 blocks following the opening of a position, you will need to modify the code accordingly:

    block.timestamp > start + 7 days

You will now have to do something like

    block.number > start_block_number + numberOfBlocksIn7Days

To calculate the number of blocks in 7 days, you can use an average block time for Ethereum which is approximately 15 seconds per block. There are 86400 seconds in a day, so for 7 days, we have:

    7 * 86400 seconds = 604800 seconds

Now, we can calculate the average number of blocks in 7 days:

    604800 seconds / 15 seconds per block = 40320 blocks

So your updated condition would look like:

    block.number > start_block_number + 40320

Miners can manipulate the block timestamp to some extent (up to around 900 seconds in the future, or 15 minutes), which could affect the contract's behavior if it heavily relies on precise timekeeping. 

The usage of *block.timestamp* can be found in all the files of this project. Therefore, it will only be mentioned here but it is present in basically all files.

# **Position.sol**

## 1.Inefficient placement of require statement in the *initializeClone* function

The require statement checking the price condition can be moved before the *setOwner* function call. The current placement could lead to unnecessary gas fees in case the function reverts on the price condition:

    if(_coll < minimumCollateral) revert InsufficientCollateral();
    price = _mint * ONE_DEC18 / _coll;
    if (price > _price) revert InsufficientCollateral();
    setOwner(owner);

# **MintingHub.sol**

## 1.Add require statement for allowance checking

In several functions throughout the file, the comment sections explicitly mention the necessity to set allowances before executing the function. This precondition is essential for the successful execution of numerous fund transfer operations. Nevertheless, if a user inadvertently omits setting the required allowances, the function will revert upon reaching the transfer calls, causing the user to incur unnecessary gas fees.

To mitigate this issue, we recommend incorporating require statements at the beginning of the relevant functions to validate the allowances. For instance, in the openPosition function, adding the following require statements will ensure that the allowances are checked before executing the transfers:

    require(IERC20(zchf).allowance(msg.sender, address(this)) >= OPENING_FEE,"ZCHF allowance not set for opening fee");
    require(IERC20(_collateralAddress).allowance(msg.sender, address(this)) >= _initialCollateral, "Collateral allowance not set for initial collateral"
);

Several other functions where such a require statement should be added are *clonePosition*, *launchChallenge*, **

## 2.Inefficient placement of require statements

In the *openPosition* function, the require statement is currently positioned at the end of the function. To prevent users from incurring unnecessary gas fees if the function fails to meet the condition specified in the require statement, it should be relocated to the beginning of the function:

     require(_initialCollateral >= _minCollateral, "must start with min col");
     IPosition pos = IPosition(
            POSITION_FACTORY.createNewPosition(
                msg.sender,
                address(zchf),
                _collateralAddress,
                _minCollateral,
                _mintingMaximum,
                _initPeriodSeconds,
                _expirationSeconds,
                _challengeSeconds,
                _mintingFeePPM,
                _liqPrice,
                _reservePPM
            )
        );
        
Similarly, in the *splitChallenge* function, two require statements are placed in the middle of the function. To avoid potential unnecessary gas fees for the user, these statements should be moved to the beginning of the function:

    Challenge storage challenge = challenges[_challengeNumber];
    require(challenge.challenger != address(0x0));
    
    uint256 min = IPosition(challenge.position).minimumCollateral();
    require(challenge.size >= min);
    require(copy.size >= min);

    Challenge memory copy = Challenge(
            challenge.challenger,
            challenge.position,
            splitOffAmount,
            challenge.end,
            challenge.bidder,
            (challenge.bid * splitOffAmount) / challenge.size
        );

    ...
