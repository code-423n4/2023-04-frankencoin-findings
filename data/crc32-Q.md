## 1. The 3% threshold should be higher
According to the docs: `Shareholders with at least 3% of the votes gain veto power.`
```
function denyMinter(address _minter, address[] calldata _helpers, string calldata _message) override external {
      if (block.timestamp > minters[_minter]) revert TooLate();
      reserve.checkQualified(msg.sender, _helpers);
      delete minters[_minter];
      emit MinterDenied(_minter, _message);
}
```
A malicious share holder with 3% of the votes can deny all of the proposals and seems it should be a higher number.

## 2. `StablecoinBridge` should give back the allowance when reaches to the `limit`
Proof of Concept (create a new test file and paste the following code):
```
// @ts-nocheck
import {expect} from "chai";
import { floatToDec18, dec18ToFloat } from "../scripts/math";
const { ethers, network } = require("hardhat");
const BN = ethers.BigNumber;
import { createContract } from "../scripts/utils";

let ZCHFContract, accounts;
let owner;
let alice;
let bob
let mockXCHF;
let bridge;

describe("Testing allowance value", () => {

    it("allowance value won't be set to 0 while the bridge limit is reached", async () => {
        accounts = await ethers.getSigners();
        owner = accounts[0];
        alice = accounts[1];
        bob = accounts[2];

        // We assume the limit is 1000
        let limit : BigNumber = floatToDec18(1000);

        ZCHFContract = await createContract("Frankencoin", [10 * 86_400]);
        mockXCHF = await createContract("TestToken", ["CryptoFranc", "XCHF", 18]);
        bridge = await createContract("StablecoinBridge", [mockXCHF.address, ZCHFContract.address, limit]);

        let applicationPeriod = BN.from(0);
        let applicationFee = BN.from(0);
        let msg = "XCHF Bridge"
        await expect(ZCHFContract.suggestMinter(bridge.address, applicationPeriod, 
            applicationFee, msg)).to.emit(ZCHFContract, "MinterApplied");
        await ethers.provider.send('evm_increaseTime', [60]); 
        await network.provider.send("evm_mine") 
        let isMinter = await ZCHFContract.isMinter(bridge.address);
        expect(isMinter).to.be.true;

        /* --------- Scenario simulation --------------- */

        // step1: alice has 1000 XCHF tokens and bob has 200 XCHF tokens
        let amount = floatToDec18(1000);
        await mockXCHF.mint(alice.address, amount);
        let balanceOfAliceXCHF = await mockXCHF.balanceOf(alice.address);
        expect(balanceOfAliceXCHF).to.be.equal(amount);
        let amount2 = floatToDec18(200);
        await mockXCHF.mint(bob.address, amount2);
        let bobInitialXCHFBalance = await mockXCHF.balanceOf(bob.address);
        expect(bobInitialXCHFBalance).to.be.equal(amount2);


        // step2: alice swaps all of the 1000 XCHF tokens for ZCHF tokens
        await mockXCHF.connect(alice).approve(bridge.address, amount);
        expect(await mockXCHF.allowance(alice.address, bridge.address)).to.be.equal(amount);
        await bridge.connect(alice)["mint(uint256)"](amount);
        expect(await ZCHFContract.balanceOf(alice.address)).to.be.equal(amount);
        expect(await mockXCHF.allowance(alice.address, bridge.address)).to.be.equal(floatToDec18(0));

        // step3: now limit is reached
        expect(await bridge.limit()).to.be.equal(limit);
        expect(await mockXCHF.balanceOf(bridge.address)).to.be.equal(limit);

        // step4: bob wants to swap XCHF tokens and gets limit error
        await mockXCHF.connect(bob).approve(bridge.address, bobInitialXCHFBalance);
        expect(await mockXCHF.allowance(bob.address, bridge.address)).to.be.equal(bobInitialXCHFBalance);
        await expect(
            bridge.connect(bob)["mint(uint256)"](bobInitialXCHFBalance)
        ).to.be.revertedWith("limit");

        // step5: bob approved value for bridge is not set to 0
        expect(await mockXCHF.allowance(bob.address, bridge.address)).to.be.equal(bobInitialXCHFBalance);
        expect(await mockXCHF.allowance(bob.address, bridge.address)).not.to.be.equal(floatToDec18(0));
        expect(await mockXCHF.allowance(bob.address, bridge.address)).greaterThan(floatToDec18(0));
    });

});

``` 
Recommendation:
Step 5 should be implemented (set allowance value to 0 when bridge limit is reached)