# Installation & Building

There are several ways to install ItyFuzz. We recommend using the [ityfuzzup](installation-and-building.md#ityfuzzup-recommended) script.

### ityfuzzup **(Recommended)**

ityfuzzup is a script that automatically installs all dependencies and installs ItyFuzz.

```bash
curl -L https://ity.fuzz.land/ | bash
ityfuzzup
```

To update ItyFuzz, run `ityfuzzup` again.

### Build from Source

You first need to install Rust through https://rustup.rs/

You need to have `libssl-dev` (OpenSSL) and `libz3-dev` installed on your system. On Linux, you probably also need to install Clang >= 12.0.0 and use it for `CC` environment variables.

```bash
# Ubuntu:
sudo apt install libssl-dev libz3-dev pkg-config cmake build-essential clang
# macOS:
brew install openssl z3
```

Then, you can use `cargo` to build ItyFuzz. This can take seconds to hours depending on the amount of your CPU cores and memory.

```bash
git clone https://github.com/fuzzland/ityfuzz.git && cd ityfuzz
git submodule update --recursive --init
cargo build --release
```

This would create an executable at `target/release/ityfuzz`. This executable only supports offchain fuzzing (i.e., fuzzing without interacting with the blockchain). To build an executable for onchain fuzzing, run:

```bash
cargo build --release --features flashloan_v2
```

If you are developing ItyFuzz, you can create a debug build (much slower than release builds) and add the following useful features, which will enable more logs and assertions:

```bash
cargo build --features flashloan_v2,debug,flashloan_debug
```
