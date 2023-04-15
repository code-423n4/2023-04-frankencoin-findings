**contracts/Position.sol**
- L104 - The NotQualified() error is created, but it is never used, this generates unnecessary gas waste in the deploy that can be avoided.

- L159 - With the implementation done, if price == newPrice, else would be executed anyway, this is unnecessary, therefore a better implementation could be:	if(newPrice != price){
            if (newPrice > price) restrictMinting(3 days);
            else checkCollateral(collateralBalance(), newPrice);

            price = newPrice;
            emitUpdate(); 
        }


**contracts/MintingHub.sol**
- L5/7 - IReserve and Ownable are imported, but they are never used, therefore, when performing the deploy, an unnecessary gas expense is generated and the code is less understandable when seeing all the complete code. It would be advisable to eliminate the imports.

- L181/188 - Public Functions Not Called By The Contract Should Be Declared External Instead
In this specific case minBid() has two functions, one public and the other internal, this is a waste of gas, deploy and an unnecessary amount of complexity, the simplest thing would be to do just one public function or one external and another internal.


**contracts/PositionFactory.sol**
- L5 - IFrankencoin is imported, but it is never used, therefore it should be eliminated so as not to generate an extra expense in the deploy and less understanding when seeing all the complete code.


**contracts/StablecoinBridge.sol**
- L5 - IERC677Receiver is imported, but it is never used, therefore it should be removed so as not to generate extra expense in the deploy and less understanding when seeing all the complete code.


**contracts/Frankencoin.sol**
- L25 - A constant is created in storage, but it is only used in the suggestMinter() function, therefore it could be created only locally in that function, this would generate less gas cost.

- L144/286 - The balance - minReserve operation could be unchecked since the pre-check is done in L141, therefore we could save gas by avoiding this check.
The same happens with _amount - reserveLeft, with the precheck in L282.


**contracts/Equity.sol**
- L7 - IERC677Receiver is imported, but it is never used, therefore it should be removed so as not to generate extra expense in the deploy and less understanding when seeing all the complete code.

- L39/46/59 - A constant is created in storage, but they are only used in the price(), checkQualified() and canRedeem() functions, therefore they could be created only locally in those functions, this would generate less cost of gas.

- L294 - The operation totalShares - shares could be unchecked since in L293 the pre-check is done, therefore we could save gas by avoiding this check.
