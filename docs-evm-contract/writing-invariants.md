# Writing Invariants

ItyFuzz, by default, supports finding [Echidna](https://github.com/crytic/echidna) and [Scribble](https://docs.scribble.codes/) invariant violations. You do not need to change the code if it works for Echidna, Mythril, Harvey, etc. ItyFuzz also supports [`bug()`](writing-invariants.md#bug-support), which indicates the current code shall not be reached.&#x20;

### Echidna Support

Any contracts bearing functions starting with `echidna_` will be treated as invariants and will be tested by ItyFuzz. If it returns `false`, the fuzzer will report a bug.

```solidity
function echidna_test() public {
    assert(false);
}
```

### Scribble Support

Scribble is a tool for writing specifications for Solidity contracts. ItyFuzz supports Scribble annotations after it is compiled by `scribble`.

For example, the following contract has a Scribble annotation that specifies the return value of `inc`:

```bash
contract Foo {
    /// #if_succeeds {:msg "P1"} y == x + 2;
    function inc(uint x) public pure returns (uint y) {
        return x+1;
    }
}
```

You need to compile the contract using `scribble` and pass the compiled contract to ItyFuzz

Note that you must add `--no-assert` to the `scribble` command. Otherwise, ItyFuzz will not detect any bugs.

```bash
scribble test.sol --output-mode flat --output compiled.sol --no-assert
```

Then compile with `solc` and run ItyFuzz:

```bash
solc compiled.sol --bin --abi --overwrite -o build
ityfuzz evm -t "build/*" [More Arguments]
```

### `bug()` Support

You can insert `bug()` or `typed_bug(string message)` in your contract to report a condition when the bug is found.

For instance, a simple case can be written as follows:

```solidity
function buy_token() public {
    if (msg.sender != owner) {
        bug();
    }
}
```

The implementation of `bug()` is as follows:

```solidity
library FuzzLand {
    event AssertionFailed(string message);
  
    function bug() internal {
        emit AssertionFailed("Bug");
    }
  
    function typed_bug(string memory data) internal {
        emit AssertionFailed(data);
    }

}

function bug()  {
    FuzzLand.bug();
}

function typed_bug(string memory data)  {
    FuzzLand.typed_bug(data);
}
```

You can either paste the code above into your contract or import it from [`solidity_utils/lib.sol`](https://github.com/fuzzland/ityfuzz/blob/master/solidity\_utils/lib.sol), if you are using `bug` or `typed_bug`.
