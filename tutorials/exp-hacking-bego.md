# \[Exp] Hacking BEGO

BEGO on Binance Smart Chain has a bug in the contract allowing arbitrary mint.

Full exploit: [https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/BEGO\_exp.sol](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/BEGO\_exp.sol)

### Using ItyFuzz to Solve

BEGO contract that is vulnerable:

* [0xc342774492b54ce5F8ac662113ED702Fc1b34972](https://bscscan.com/address/0xc342774492b54ce5F8ac662113ED702Fc1b34972)

The contracts are exploitable before block number 22315678. We'll fork the chain at block number 22315678 and let ItyFuzz find the exploit.

To conduct an ItyFuzz campaign, run the following command:

```
ityfuzz evm\
 -t 0xc342774492b54ce5F8ac662113ED702Fc1b34972\
 -f -c BSC\
 --onchain-block-number 22315678\
 --onchain-etherscan-api-key <your etherscan api key> # (Optional) specify your BSC etherscan api key
```
