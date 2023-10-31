# \[CTF] Verilog CTF (Offchain)

### Introduction

Source code of the challenge: [https://github.com/Verilog-Solutions/CTF-2022-wMaticV2](https://github.com/Verilog-Solutions/CTF-2022-wMaticV2)

The project consists of a buggy WMATICv2 token contract containing a function `redeem` allowing reentrancy attack. Additionally, there is a `Bounty.sol` contract, which rewards the first person who successfully calls its `getBounty` function.

There are two trackers of the total supply of WMATICv2 tokens and our goal is to successfully call `getBounty`, which requires these two trackers to be significantly deviated.

Below is the intended solution that conducts flash loan and reentrancy attacks:

```
0. Borrow k MATIC such that k > balance() / 10
1. depositMATIC() with k MATIC
2. redeem(k * 1e18) -- reentrancy contract --> getBounty()
3. Return k MATIC
```

### Using ItyFuzz to Solve

We have modified some code to make it easier to deploy: [https://ityfuzz.assets.fuzz.land/verilog.zip](https://ityfuzz.assets.fuzz.land/verilog.zip)

**Step 1: Add Invariants**

Download the above contracts and modify the contracts to insert an invariant, determining whether the logic breaks. Here, the logic holds if the two trackers are equal and break otherwise.

Modify `Bounty.sol` as follows:

```solidity
...

+ event AssertionFailed(string message);

function getBounty() public returns (bool) {
    uint256 delta = WMATICV2.totalSupply() >= WMATICV2.balance()
        ? WMATICV2.totalSupply() - WMATICV2.balance()
        : WMATICV2.balance() - WMATICV2.totalSupply();
    uint256 tolerance = WMATICV2.balance() / 10;
    if (delta > tolerance) {
        // reward the first finder
        isHacked = true;
+       emit AssertionFailed("we hacked it!");

-       IERC20(WMATIC).transfer(msg.sender, IERC20(WMATIC).balanceOf(address((this))));
        winner = address(msg.sender);
    }
    return isHacked;
}

```

We inserted an event `AssertionFailed` to indicate that the invariant is broken and the challenge is solved. ItyFuzz uses `AssertionFailed(string)` events to determine whether the challenge is solved.

We also remove the transfer of WMATIC token to the winner since we will not deploy the WMATIC token contract. Instead, we will use ItyFuzz to simulate the execution of the contract and determine whether the invariant is broken.

**Step 2: Build**

Ensure you have installed `solc >= 0.8.0`. Then, run the following command to build the contracts:

```
solc *.sol -o . --bin --abi --overwrite
```

**Step 3: Run ItyFuzz**

Use the following command to run ItyFuzz:

```
ityfuzz evm -f -t "[YOUR BUILD DIRECTORY]/*"
```

The above command will run ItyFuzz in offchain mode and specify the target contracts (`-t`). `-f` enables the fuzzer to conduct flash loan operations. It will automatically detect the invariant violation and generate exploits in less than 10 seconds.
