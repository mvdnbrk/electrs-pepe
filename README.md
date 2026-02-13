# Pepecoin Esplora - Electrs backend API

A block chain index engine and HTTP API written in Rust, ported to Pepecoin based on [Blockstream/electrs](https://github.com/blockstream/electrs).

Used as the backend for the Pepecoin Esplora block explorer.

### Installing & indexing

Install Rust, Pepecoin Core (no `txindex` needed) and the `clang` and `cmake` packages, increase maximum number open files by `ulimit -n 100000` and then:

```bash
$ git clone https://github.com/mvdnbrk/electrs-pepe && cd electrs-pepe
$ git checkout pepecoin
$ cargo run --release --bin electrs -- -vvvv --daemon-dir ~/.pepecoin
```

Note that our indexes are incompatible with Bitcoin electrs's and must be created separately.

Creating the indexes should take a few hours on a beefy machine with SSD.

### Light mode

For personal or low-volume use, you may set `--lightmode` to reduce disk storage requirements
by roughly 50% at the cost of slower and more expensive lookups.

With this option set, raw transactions and metadata associated with blocks will not be kept in rocksdb
(the `T`, `X` and `M` indexes),
but instead queried from pepecoind on demand.

### Notable changes from Electrs:

- Ported to Pepecoin (Magic bytes: `0xe4b8b8a4`, Genesis Hash: `6926978583488737b9875e53381a806955dfa503893603417643b2f671c8907f`).
- Support for Pepecoin address prefixes (starting with `P`, Base58 prefix 56).
- HTTP REST API in addition to the Electrum JSON-RPC protocol, with extended transaction information.

- Extended indexes and database storage for improved performance under high load:

  - A full transaction store mapping txids to raw transactions is kept in the database under the prefix `t`.
  - An index of all spendable transaction outputs is kept under the prefix `O`.
  - An index of all addresses (encoded as string) is kept under the prefix `a` to enable by-prefix address search.
  - A map of blockhash to txids is kept in the database under the prefix `X`.
  - Block stats metadata (number of transactions, size and weight) is kept in the database under the prefix `M`.

  With these new indexes, pepecoind is no longer queried to serve user requests and is only polled
  periodically for new blocks and for syncing the mempool.

### CLI options

In addition to electrs's original configuration options, a few new options are also available:

- `--http-addr <addr:port>` - HTTP server address/port to listen on (default: `127.0.0.1:3002`).
- `--electrum-rpc-addr <addr:port>` - Electrum server address/port to listen on (default: `127.0.0.1:50002`).
- `--daemon-rpc-addr <addr:port>` - Pepecoin daemon RPC address/port (default: `127.0.0.1:33873`).
- `--lightmode` - enable light mode (see above)
- `--cors <origins>` - origins allowed to make cross-site request (optional, defaults to none).
- `--address-search` - enables the by-prefix address search index.
- `--index-unspendables` - enables indexing of provably unspendable outputs.
- `--utxos-limit <num>` - maximum number of utxos to return per address.
- `--electrum-txs-limit <num>` - maximum number of txs to return per address in the electrum server (does not apply for the http api).
- `--electrum-banner <text>` - welcome banner text for electrum server.

Additional options with the `liquid` feature:
- `--parent-network <network>` - the parent network this chain is pegged to.

Additional options with the `electrum-discovery` feature:
- `--electrum-hosts <json>` - a json map of the public hosts where the electrum server is reachable, in the [`server.features` format](https://electrumx.readthedocs.io/en/latest/protocol-methods.html#server.features).
- `--electrum-announce` - announce the electrum server on the electrum p2p server discovery network.

See `$ cargo run --release --bin electrs -- --help` for the full list of options.

## License

MIT
