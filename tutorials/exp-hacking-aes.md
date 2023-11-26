# \[Exp] Hacking AES

AES on Binance Smart Chain has experienced a price manipulation attack requiring flash loan. It is one of the most complex attacks we have seen so far.

Full exploit: [https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/AES\_exp.sol](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/src/test/AES\_exp.sol)

### Using ItyFuzz to Solve

AES LP contract that is vulnerable:

* [0x40eD17221b3B2D8455F4F1a05CAc6b77c5f707e3](https://bscscan.com/address/0x40eD17221b3B2D8455F4F1a05CAc6b77c5f707e3)

The contracts are exploitable before block number 23695904. We'll fork the chain at block number 23695904 and let ItyFuzz find the exploit.

To conduct an ItyFuzz campaign, run the following command:

```bash
ityfuzz evm\
 -t 0x40eD17221b3B2D8455F4F1a05CAc6b77c5f707e3\
 -f -c BSC\
 --onchain-block-number 23695904\
 --onchain-etherscan-api-key <your etherscan api key> # (Optional) specify your BSC etherscan api key

```
