# node-launch

```
git clone https://github.com/Snapchain/node-launch.git
cd node-launch/sepolia
cp .env.example .env
```

then set `ERIGON_DATA_DIR` and `PRYSM_DATA_DIR` to the locations inside the mounted volume

run `sudo mkdir -p` to create those directories

## Overview

Launch [Erigon](https://github.com/ledgerwatch/erigon) and [Prysm](https://github.com/prysmaticlabs/prysm) to provide L1 RPC of Testnet [Sepolia](https://sepolia.etherscan.io/).

## System requirements

The Sepolia node has been tested on: 8 GB Memory / 2 Intel vCPUs / 160 GB Disk + 1000 GB / SGP1 - Debian 12 x64 (DigitalOcean)

## Generate JWT token

The HTTP connection between the beacon node and execution node needs to be authenticated using a JWT token. Refer to the Prysm [authentication](https://docs.prylabs.network/docs/execution-node/authentication) for more details.

Use OpenSSL to create the token via command:

```
openssl rand -hex 32 | tr -d "\n" > "jwt.hex"
```

## Set up permission

Run the following command to avoid permission issue like [this](https://github.com/ledgerwatch/erigon/issues/3950)
```
sudo chown 1000:1000 <ERIGON_DATA_DIR>
```

## Launch L1 node

```
docker compose up -d
```

To check logs:

```
docker compose logs
```

To test the HTTP JSON-RPC:

```
curl -X POST -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[], "id":1}' http://127.0.0.1:8545
```

```
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_chainID","params":[],"id":1}' http://127.0.0.1:8545
```

```
curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["0x0", false],"id":1}' http://127.0.0.1:8545
```

To test the Beacon API:
```
curl -H "Content-Type: application/json" http://127.0.0.1:3500/eth/v1/beacon/headers
```

To convert hex number in the result to decimal, we can do `printf "%d\n" 0x<xxx>`
