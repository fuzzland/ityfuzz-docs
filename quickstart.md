# Quickstart

ItyFuzz supports offchain (local) fuzzing and onchain (fork) fuzzing for EVM and Move smart contracts.&#x20;

## Onchain Fuzzing (EVM)

To run an onchain fuzzing campaign, specify the target contract and the chain to fork.

<pre class="language-bash"><code class="lang-bash"><strong># -t [TARGET_ADDR]: specify the target contract
</strong># --onchain-block-number [BLOCK]: fork the chain at block number [BLOCK]
# -c [CHAIN_TYPE]: specify the chain

ityfuzz evm\
    -t [TARGET_ADDR]\
    --onchain-block-number [BLOCK]\
    -c [CHAIN_TYPE]\
    --onchain-etherscan-api-key [Etherscan API Key] # (Optional) specify your etherscan api key
</code></pre>

For example, to run an onchain fuzzing campaign on Ethereum targeting WETH, run:

```bash
# -t [TARGET_ADDR]: specify the target contract
# --onchain-block-number [BLOCK]: fork the chain at block number [BLOCK]
# -c [CHAIN_TYPE]: specify the chain
# -f: (Optional) allow attack to get flashloan

ityfuzz evm\
    -o\
    -t 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2\
    --onchain-block-number 0\
    -c ETH\
    --onchain-etherscan-api-key [Etherscan API Key]\
    -f
```

ItyFuzz would pull the ABI of the contract from Etherscan and fuzz it. If ItyFuzz encounters an unknown slot in the memory, it will pull the slot from chain RPC. If ItyFuzz encounters calls to an external unknown contract, it will pull the bytecode and ABI of that contract. If its ABI is unavailable, ItyFuzz will decompile and get the ABI.

## Offchain Fuzzing (EVM)

To run a local fuzzing campaign, specify the target contract (only needs bytecode and ABIs).

```bash
# -t [BUILD DIRECTORY GLOB]: specify the targets directory
# -f: (Optional) allow attack to get flashloan
# --concolic: (Optional) enable concolic execution
# --concolic-caller: (Optional) enable concolic execution to change caller to anyone

ityfuzz evm\
    -t "[BUILD DIRECTORY GLOB]"\
    -f\
    --concolic --concolic-caller
```

For example, run a simple fuzzing campaign on a compiled single contract:

```bash
ityfuzz evm -t './build/*'
```

ItyFuzz would attempt to deploy all artifacts in the directory to a blockchain with no other smart contracts.

Specifically, the project directory should contain a few `[X].abi` and `[X].bin` files. For example, to fuzz a contract named `main.sol`, you should ensure `main.abi` and `main.bin` exist in the project directory. The fuzzer will automatically detect the contracts in the directory and the correlation between them (see [`tests/evm/multi-contract`](https://github.com/fuzzland/ityfuzz/tree/master/tests/evm/multi-contract)), and fuzz them.

Optionally, if ItyFuzz fails to infer the correlation between contracts, you can add a `[X].address`, where `[X]` is the contract name to specify the address of the contract.

To define a custom invariant, check out Custom Invariant or Echidna / Scribble Support.

**Caveats:**

* Remember that ItyFuzz is fuzzing on a clean blockchain, so you should ensure all related contracts (e.g., ERC20 token, Uniswap, etc.) are deployed to the blockchain before fuzzing.
* If your smart contract requires constructor arguments, please refer to the Constructor Arguments section.

## Offchain Fuzzing (MoveVM)

> Move support is still under development. Please contact us if you want to try it out.

Compile the contracts with `sui move build` and run ItyFuzz:

```bash
# build example contract that contains a bug
cd ./tests/move/share_object
sui move build

# get back to ItyFuzz and run fuzzing on the built contract
cd ../../../
ityfuzz move -t "./tests/move/share_object/build"

```

#### Defining Invariants

You can emit an event of `AAAA__fuzzland_move_bug` in your contract to report a condition when the bug is found.

```rust
// define the event struct
use sui::event;

struct AAAA__fuzzland_move_bug has drop, copy, store {
    info: u64
}

... 
    // inside function
    event::emit(AAAA__fuzzland_move_bug { info: 1 });
...

```

An example contract that reports a bug can be found in [`tests/move/share_object/sources/test.move`](https://github.com/fuzzland/ityfuzz/tree/master/tests/move/share\_object).
