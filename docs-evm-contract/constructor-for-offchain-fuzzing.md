# Constructor for Offchain Fuzzing

When fuzzing a contract offchain, you may need to provide constructor arguments to initialize the contract state. ItyFuzz provides two methods to pass in constructor arguments. These arguments are necessary for initializing the state of the contract when deployed.

**Method 1: CLI Arguments**

The first method is to pass in the constructor arguments directly as CLI arguments.

When you run ItyFuzz using the CLI, you can include the `--constructor-args` flag followed by a string that specifies the arguments for each constructor.

The format is as follows:

```bash
ityfuzz evm -t 'build/*'\
    --constructor-args "ContractName:arg1,arg2,...;AnotherContract:arg1,arg2,..;"
```

For example, if you have two contracts, `main` and `main2`, both having a `bytes32` and a `uint256` as constructor arguments, you would pass them in like this:

```bash
ityfuzz evm -t 'build/*'\
    --constructor-args "main:1,0x6100000000000000000000000000000000000000000000000000000000000000;main2:2,0x6200000000000000000000000000000000000000000000000000000000000000;"
```

**Method 2: Server Forwarding**

The second method is to use our server to forward requests to a user-specified RPC, and cli will fetch the constructor arguments from the transactions sent to the RPC.

Firstly, go to the `/server` directory, and install the necessary packages:

```bash
cd /server
npm install
```

Then, start the server using the following command:

```bash
node app.js
```

By default, the server will forward requests to `http://localhost:8545`, which is the default address for [Ganache](https://github.com/trufflesuite/ganache), if you do not have a local blockchain running, you can use Ganache to start one. If you wish to forward requests to another location, you can specify the address as a command-line argument like so:

```bash
node app.js http://localhost:8546
```

Once the server is running, you can deploy your contract to `localhost:5001` using a tool of your choice.

For example, you can use Foundry to deploy your contract through the server:

```bash
forge create src/flashloan.sol:main2 --rpc-url http://127.0.0.1:5001 --private-key 0x0000000000000000000000000000000000000000000000000000000000000000 --constructor-args "1" "0x6100000000000000000000000000000000000000000000000000000000000000"
```

Finally, you can fetch the constructor arguments using the `--fetch-tx-data` flag:

```bash
ityfuzz evm -t 'tests/evm/multi-contract/*' --fetch-tx-data
```

ItyFuzz will fetch the constructor arguments from the transactions forwarded to the RPC through the server.
