## Installation

Install [latest Rust](https://rustup.rs/) (1.31+),
[latest Pepecoin Core](https://github.com/pepecoinppc/pepecoin) (0.16+)
and a compatible Electrum wallet.

Also, install the following packages (on Debian):
```bash
$ sudo apt update
$ sudo apt install clang cmake build-essential  # for building 'rust-rocksdb'
```

## Build

First build should take ~20 minutes:
```bash
$ cargo build --release
```


## Pepecoind configuration

Allow Pepecoin daemon to sync before starting Electrum server:
```bash
$ pepecoind -server=1 -txindex=0 -prune=0
```

If you are using `-rpcuser=USER` and `-rpcpassword=PASSWORD` for authentication, please use `--cookie="USER:PASSWORD"` command-line flag.
Otherwise, `~/.pepecoin/.cookie` will be read, allowing this server to use pepecoind JSONRPC interface.

## Usage

First index sync should take a few hours depending on hardware:
```bash
$ cargo run --release --bin electrs -- -vvv --timestamp --db-dir ./db [--cookie="USER:PASSWORD"]
...
2026-02-12T18:27:42 - INFO - RPC server running on 127.0.0.1:50002
```

The index database is stored here:
```bash
$ du db/
```

## Electrum client
```bash
# Connect only to the local server, for better privacy
$ ./scripts/local-electrum.bash
+ ADDR=127.0.0.1
+ PORT=50002
+ PROTOCOL=t
+ electrum --oneserver --server=127.0.0.1:50002:t
```

In order to use a secure connection, TLS-terminating proxy (e.g. [hitch](https://github.com/varnish/hitch)) is recommended:
```bash
$ hitch --backend=[127.0.0.1]:50002 --frontent=[127.0.0.1]:50003 pem_file
$ electrum --oneserver --server=127.0.0.1:50003:s
```

## Docker
```bash
$ docker build -t electrs-app .
$ docker run --network host \
             --volume $HOME/.pepecoin:/home/user/.pepecoin:ro \
             --volume $PWD:/home/user \
             --rm -i -t electrs-app
```

## Monitoring

Indexing and serving metrics are exported via [Prometheus](https://github.com/pingcap/rust-prometheus):

```bash
$ sudo apt install prometheus
$ echo "
scrape_configs:
  - job_name: electrs
    static_configs:
    - targets: ['localhost:4224']
" | sudo tee -a /etc/prometheus/prometheus.yml
$ sudo systemctl restart prometheus
$ firefox 'http://localhost:9090/graph?g0.range_input=1h&g0.expr=index_height&g0.tab=0'
```
