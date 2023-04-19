#### Gas Optimizations
|       | Issue                      | Instances | Estimated gas(deployments) | Estimated gas(avg method call) |
| :---: | :------------------------- | :-------: | :------------------------: | :----------------------------: |
|   1   | Optimize revert statements |     1     |            4603            |               62               |
|   2   | Optimize boolean logic     |     1     |            3003            |               11               |
|   3   | Cache function call        |     1     |            -400            |              117               |

**Total: 3 instances over 3 issues**

---

## 001. Optimize revert statements

#### PoC:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "../../contracts/Frankencoin.sol";
import "forge-std/Test.sol";

contract _001Gas is Test {
    Frankencoin internal immutable zchf;
    address internal immutable TEST_MINTER =
        vm.addr(uint256(bytes32("TEST_MINTER")));
    uint256 internal constant TEST_MIN_APPLICATION_PERIOD = 864000;

    constructor() {
        zchf = new Frankencoin(TEST_MIN_APPLICATION_PERIOD);
    }

    function test_suggestMinter() public {
        zchf.suggestMinter(address(0), 0, 0, "");
        
        vm.prank(address(0));
        zchf.mint(TEST_MINTER, zchf.MIN_FEE());

        vm.startPrank(TEST_MINTER);
        zchf.suggestMinter(
            TEST_MINTER,
            zchf.MIN_APPLICATION_PERIOD(),
            zchf.MIN_FEE(),
            ""
        );
        vm.stopPrank();
    }
}
```

#### Fix:

```diff
    function suggestMinter(address _minter, uint256 _applicationPeriod, uint256 _applicationFee, string calldata _message) override external {
-      if (_applicationPeriod < MIN_APPLICATION_PERIOD && totalSupply() > 0) revert PeriodTooShort();
-      if (_applicationFee < MIN_FEE  && totalSupply() > 0) revert FeeTooLow(); // @audit-gas double check for totalSupply > 0
       if (minters[_minter] != 0) revert AlreadyRegistered();
+      if (totalSupply() > 0) {
+         if (_applicationPeriod < MIN_APPLICATION_PERIOD) revert PeriodTooShort();
+         if (_applicationFee < MIN_FEE) revert FeeTooLow(); // @audit-gas double check for totalSupply > 0
+      }
```

##### Before:

| contracts/Frankencoin.sol:Frankencoin contract |                 |     |        |     |         |
| ---------------------------------------------- | --------------- | --- | ------ | --- | ------- |
| Deployment Cost                                | Deployment Size |     |        |     |         |
| 2960396                                        | 15168           |     |        |     |         |
| Function Name                                  | min             | avg | median | max | # calls |
...
| suggestMinter                                  | 34804           | 36783 | 36783  | 38762 | 2       |

##### After:

| contracts/Frankencoin.sol:Frankencoin contract |                 |     |        |     |         |
| ---------------------------------------------- | --------------- | --- | ------ | --- | ------- |
| Deployment Cost                                | Deployment Size |     |        |     |         |
| 2955793                                        | 15145           |     |        |     |         |
| Function Name                                  | min             | avg | median | max | # calls |
...
| suggestMinter                                  | 34601           | 36721 | 36721  | 38842 | 2       |

Difference (Before-After):

Deployment Cost: 4603
min: 203 avg: 62 median: 62 max: -80

---

## 002. Optimize boolean logic

#### PoC:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "forge-std/console2.sol";

import "../../contracts/Frankencoin.sol";

contract _002Gas is Test {
    Frankencoin internal immutable zchf;

    address internal immutable TEST_MINTER =
        vm.addr(uint256(bytes32("TEST_MINTER")));

    address internal immutable USER_1 = vm.addr(uint256(bytes32("USER_1")));
    address internal immutable USER_2 = vm.addr(uint256(bytes32("USER_2")));

    uint256 internal constant TEST_MIN_APPLICATION_PERIOD = 864000;

    constructor() {
        zchf = new Frankencoin(TEST_MIN_APPLICATION_PERIOD);
    }

    function test_allowance() public {
        zchf.suggestMinter(TEST_MINTER, 0, 0, "");
        vm.startPrank(TEST_MINTER);
        zchf.mint(USER_1, 1e30);
        vm.stopPrank();
        vm.startPrank(USER_1);
        zchf.approve(USER_2, 1e30);
        vm.stopPrank();

        zchf.allowance(USER_1, TEST_MINTER);
        zchf.allowance(USER_1, USER_2);
        zchf.allowance(USER_2, USER_1);
    }
}
```

#### Fix:

```diff
    function allowanceInternal(address owner, address spender) internal view override returns (uint256) {
       uint256 explicit = super.allowanceInternal(owner, spender);
-      if (explicit > 0){
-         return explicit; // don't waste gas checking minter
-      } else if (isMinter(spender) || isMinter(isPosition(spender))){
-         return INFINITY;
-      } else {
-         return 0;
-      } // @audit-gas Can be optimized by reducing if else
+      if (explicit == 0) {
+         if (isMinter(spender) || isMinter(isPosition(spender))){
+            return INFINITY;
+         }
+      }
+      return explicit; // don't waste gas checking minter
    }
```

##### Before:

| contracts/Frankencoin.sol:Frankencoin contract |                 |      |        |      |         |
| ---------------------------------------------- | --------------- | ---- | ------ | ---- | ------- |
| Deployment Cost                                | Deployment Size |      |        |      |         |
| 2960396                                        | 15168           |      |        |      |         |
| Function Name                                  | min             | avg  | median | max  | # calls |
| allowance                                      | 892             | 4636 | 3381   | 9635 | 3       |
...

##### After:

| contracts/Frankencoin.sol:Frankencoin contract |                 |      |        |      |         |
| ---------------------------------------------- | --------------- | ---- | ------ | ---- | ------- |
| Deployment Cost                                | Deployment Size |      |        |      |         |
| 2957393                                        | 15153           |      |        |      |         |
| Function Name                                  | min             | avg  | median | max  | # calls |
| allowance                                      | 881             | 4627 | 3383   | 9617 | 3       |
...

Difference (Before-After):

Deployment Cost: 3,003
min: 11 avg: 11 median: -2 max: 18

---

## 003. Cache function call

#### PoC:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "forge-std/console2.sol";

import "../../contracts/Frankencoin.sol";

contract _003Gas is Test {
    Frankencoin internal immutable zchf;

    address internal immutable TEST_MINTER =
        vm.addr(uint256(bytes32("TEST_MINTER")));

    address internal immutable USER_1 = vm.addr(uint256(bytes32("USER_1")));

    uint256 internal constant TEST_MIN_APPLICATION_PERIOD = 864000;

    constructor() {
        zchf = new Frankencoin(TEST_MIN_APPLICATION_PERIOD);
    }

    function test_calculateAssignedReserve() public {
        zchf.suggestMinter(TEST_MINTER, 0, 0, "");
        vm.startPrank(TEST_MINTER);
        zchf.mint(USER_1, 1e30, 100_000, 100_000);
        zchf.calculateAssignedReserve(1e30, 100_000);
        zchf.balanceOf(address(zchf.reserve()));
        zchf.notifyLoss(2e29-1000);
        zchf.balanceOf(address(zchf.reserve()));
        zchf.calculateAssignedReserve(1e30, 100_000);
        zchf.notifyLoss(1000);
        zchf.calculateAssignedReserve(1e30, 100_000);
        vm.stopPrank();
    }
}
```
#### Fix:

```diff
    function calculateAssignedReserve(uint256 mintedAmount, uint32 _reservePPM) public view returns (uint256) {
       uint256 theoreticalReserve = _reservePPM * mintedAmount / 1000000; // @audit-qa cast to 1e6
       uint256 currentReserve = balanceOf(address(reserve));
-      if (currentReserve < minterReserve()){ // @audit-gas cache function call
+      uint256 minterReserve = minterReserve();
+      if (currentReserve < minterReserve){ // @audit-gas cache function call
          // not enough reserves, owner has to take a loss
-         return theoreticalReserve * currentReserve / minterReserve();
+         return theoreticalReserve * currentReserve / minterReserve;
       } else {
          return theoreticalReserve;
       }
```

##### Before:

| contracts/Frankencoin.sol:Frankencoin contract |                 |     |        |     |         |
| ---------------------------------------------- | --------------- | --- | ------ | --- | ------- |
| Deployment Cost                                | Deployment Size |     |        |     |         |
| 2960396                                        | 15168           |     |        |     |         |
| Function Name                                  | min             | avg | median | max | # calls |
...
| calculateAssignedReserve                       | 1104            | 1339  | 1457   | 1457  | 3       |
...

##### After:

| contracts/Frankencoin.sol:Frankencoin contract |                 |     |        |     |         |
| ---------------------------------------------- | --------------- | --- | ------ | --- | ------- |
| Deployment Cost                                | Deployment Size |     |        |     |         |
| 2960796                                        | 15170           |     |        |     |         |
| Function Name                                  | min             | avg | median | max | # calls |
...
| calculateAssignedReserve                       | 1122            | 1222  | 1272   | 1272  | 3       |
...


Difference (Before-After):

Deployment Cost: -400
min: -18 avg: 117 median: 185 max: 185

