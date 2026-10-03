# Building the Undercroft daemon on Linux

This is the SHA256d Bitcoin Core v25.0 fork behind The Undercroft (UCFT).
It's headless by design -- the GUI (bitcoin-qt) isn't built, since this repo
doesn't ship the Qt frontend code at all. This covers building `bitcoind`
and `bitcoin-cli` for a pool operator, a node runner, or anyone who wants a
node without the Windows wallet app.

## 1. Install build dependencies

On Ubuntu/Debian:

```bash
sudo apt-get update
sudo apt-get install -y build-essential libtool autoconf automake pkg-config \
    libboost-dev libevent-dev libsqlite3-dev python3
```

That's everything needed for a headless build with SQLite wallet support
(this fork doesn't use the older Berkeley DB wallet format, so `libdb-dev`
isn't required). If you also want UPnP/NAT-PMP port-forwarding support or
ZeroMQ notifications, add `libminiupnpc-dev libnatpmp-dev libzmq3-dev` --
neither is required to run a node.

## 2. Build

```bash
git clone https://github.com/Omletz/undercroft-core.git
cd undercroft-core
./autogen.sh
./configure --without-gui --disable-tests --disable-bench
make -j$(nproc)
```

A clean build takes roughly 10-20 minutes depending on your hardware. When
it finishes, `src/bitcoind` and `src/bitcoin-cli` are your binaries -- no
`make install` needed, just copy them wherever you want or run them
directly from `src/`.

`--disable-tests`/`--disable-bench` just skip building the test suite and
benchmarks, which you don't need to run a node and which otherwise pull in
an extra `hexdump` dependency. Drop both flags if you specifically want to
run Bitcoin Core's own test suite against this fork.

## 3. Run it

```bash
./src/bitcoind -daemon
```

Default ports: P2P 23857, RPC 23856 on mainnet (P2P 23867, RPC 23866 on
regtest). Data directory defaults to `~/.bitcoin` unless you pass
`-datadir=`.

To point the Undercroft Wallet app at this daemon instead of building its
own copy, either drop the compiled `bitcoind` into a `bin/` folder next to
`wallet_app.py` in the
[undercroft-wallet](https://github.com/Omletz/undercroft-wallet) repo (it's
found automatically, same convention as the Windows build), or just point
Settings -> Daemon location at wherever you put it.

## Verified

This exact sequence was build-tested from a genuinely blank Ubuntu 24.04
machine -- `./autogen.sh && ./configure --without-gui --disable-tests
--disable-bench && make -j$(nproc)` completed from zero with no errors,
producing working `bitcoind`/`bitcoin-cli` binaries.
