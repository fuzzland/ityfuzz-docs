# \[CTF] Verilog CTF (Onchain)

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

Here are the contracts we are interested in:

* WMATICV2: [0x5d6c48f05ad0fde3f64bab50628637d73b1eb0bb](https://polygonscan.com/address/0x5d6c48f05ad0fde3f64bab50628637d73b1eb0bb)
* Bounty: [0xbcf6e9d27bf95f3f5eddb93c38656d684317d5b4](https://polygonscan.com/address/0xbcf6e9d27bf95f3f5eddb93c38656d684317d5b4)

The contracts are exploitable before block number 35690977. We'll fork the chain at block number 35690976 and let ItyFuzz find the exploit.

To conduct an ItyFuzz campaign, run the following command:

```bash
ityfuzz evm\
 -t 0x5d6c48f05ad0fde3f64bab50628637d73b1eb0bb,0xbcf6e9d27bf95f3f5eddb93c38656d684317d5b4\
 -f -c POLYGON\
 --onchain-block-number 35690976\
 --onchain-etherscan-api-key <your etherscan api key> # (Optional) specify your Polygon etherscan api key
```

Optionally, you can also use your RPC instead of the default public RPC by setting environment variable `ETH_RPC_URL`. For instance, to use your local RPC, run:

```bash
export ETH_RPC_URL=http://polygon_rpc.xxx.com
```

After a few seconds to minutes, you should see ItyFuzz exit with the full exploit to take the fund.
