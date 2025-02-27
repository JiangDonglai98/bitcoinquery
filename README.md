# bitcoin-explorer

![rust test](https://github.com/Congyuwang/Rusty-Bitcoin-Explorer/actions/workflows/rust.yml/badge.svg)
![publish](https://github.com/Congyuwang/Rusty-Bitcoin-Explorer/actions/workflows/publish.yml/badge.svg)
[![Crates.io](https://img.shields.io/crates/v/bitcoin-explorer.svg)](https://crates.io/crates/bitcoin-explorer/)
![Downloads](https://img.shields.io/crates/d/bitcoin-explorer)
[![](https://tokei.rs/b1/github/Congyuwang/Rusty-Bitcoin-Explorer)](https://github.com/Congyuwang/Rusty-Bitcoin-Explorer)

`bitcoin_explorer` is an efficient library for reading
bitcoin-core binary blockchain file as a database (utilising multi-threading).

## Documentation

Go to [Rust Documentation](https://docs.rs/bitcoin-explorer/)

## Compatibility Note

This package deals with the binary file of another software `Bitcoin Core`.
It might not be compatible with older Bitcoin Core versions.

Tested on
`Bitcoin Core version v0.21.1.0-g194b9b8792d9b0798fdb570b79fa51f1d1f5ebaf
Copyright (C) 2009-2020 The Bitcoin Core developers`.

## Features
`db.iter_connected_block()` uses in-memory UTXO cache by default,
which requires 32GB ore more memory, but is very fast.
It can be configured to use on-disk UTXO cache for tracking unspent transactions during iterations,
which usually requires less than 1GB memory.

*By default, UTXO is stored on RAM, which requires 32GM memory
(it needs this much memory only for `db.iter_connected_block()`)*
```toml
[dependencies.bitcoin-explorer]
version = "1.2"
```

Enable on-disk UTXO cache if your memory is limited:
```toml
[dependencies.bitcoin-explorer]
version = "1.2"
features = ["on-disk-utxo"]
```

### Guide to Feature
When you have a large memory (>= 32 GB), do not use on-disk-utxo.
In-memory UTXO cache provides blazing fast speed.
Use on-disk-utxo only when your disk is limited.

## Performance

SSD allows faster performance.

Iterating through all 700000 blocks (non-connected, in sequential order) takes about 10 minutes
(Windows 10, CPU Core i7-9700, Block chain data on external SSD drive connected through USB 3.1).

With SSD, iterating through all 700000 blocks with input addresses connected takes about ? minutes.

Iterating through all 700000 blocks with input addresses connected takes about 30 minutes
using in-memory UTXO cache (by default), which requires 32GB memory

If memory is limited, iterating through all blocks with input addresses connected takes about 10 hours
using on-disk UTXO cache (with `features = ["on-disk-utxo"]`).

## Examples

### get a block (i.e., see doc for what is full/simple format)
```rust
use bitcoin_explorer::{BitcoinDB, FBlock, SBlock, Block};
use std::path::Path;

let path = Path::new("/Users/me/bitcoin");

// launch without reading txindex
let db = BitcoinDB::new(path, false).unwrap();

// get block of height 600000 (in different formats)
let block: Block = db.get_block(600000).unwrap();
let block: FBlock = db.get_block(600000).unwrap();
let block: SBlock = db.get_block(600000).unwrap();
```

### get a particular transaction (in different formats)

```rust
use bitcoin_explorer::{BitcoinDB, Transaction, FTransaction, STransaction, Txid, FromHex};
use std::path::Path;

let path = Path::new("/Users/me/bitcoin");

// !!must launch with txindex=true!!
let db = BitcoinDB::new(path, true).unwrap();

// get transaction
// e3bf3d07d4b0375638d5f1db5255fe07ba2c4cb067cd81b84ee974b6585fb468
let txid_str = "e3bf3d07d4b0375638d5f1db5255fe07ba2c4cb067cd81b84ee974b6585fb468";
let txid = Txid::from_hex(txid_str).unwrap();

// get transactions in different formats
let tx: Transaction = db.get_transaction(&txid).unwrap();
let tx: FTransaction = db.get_transaction(&txid).unwrap();
let tx: STransaction = db.get_transaction(&txid).unwrap();
```

### Iterate through blocks (in different formats)
Iterating to 700000 blocks requires about 2GB memory.
```rust
use bitcoin_explorer::{BitcoinDB, Block, SBlock, FBlock};
use std::path::Path;

let path = Path::new("/Users/me/bitcoin");

// launch without reading txindex
let db = BitcoinDB::new(path, false).unwrap();

// iterate over block from 600000 to 700000
for block in db.iter_block::<Block>(600000, 700000) {
    for tx in block.txdata {
        println!("do something for this transaction");
    }
}

// iterate over block from 600000 to 700000
for block in db.iter_block::<FBlock>(600000, 700000) {
    for tx in block.txdata {
        println!("do something for this transaction");
    }
}

// iterate over block from 600000 to 700000
for block in db.iter_block::<SBlock>(600000, 700000) {
    for tx in block.txdata {
        println!("do something for this transaction");
    }
}
```

### Iterate through blocks (in different format) with outpoints connected to outputs

Iterating to 700000 blocks with outpoints connected to outputs (with input addresses attached)
requires a minimal amount of 32GB memory.

If memory is limited, use `features = ["on-disk-utxo"]` in Cargo.toml to reduce memory use to
usually 1GB.

```rust
use bitcoin_explorer::{BitcoinDB, FConnectedBlock, SConnectedBlock};
use std::path::Path;

let path = Path::new("/Users/me/bitcoin");

// launch without reading txindex
let db = BitcoinDB::new(path, false).unwrap();
let end = db.get_max_height() as u32;

// iterate over all blocks found (simple connected format)
for block in db.iter_connected_block::<SConnectedBlock>(end) {
    for tx in block.txdata {
        println!("do something for this transaction");
    }
}
```
