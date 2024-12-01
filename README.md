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

Run the following command to avoid permission issues like
- https://github.com/ledgerwatch/erigon/issues/3950
- https://github.com/erigontech/erigon/issues/12931

```
sudo chown 100:1000 <ERIGON_DATA_DIR>
```

## Launch L1 node

```
docker compose up -d erigon prysm
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

## Set up diagnostics

First you need to build the diagnostics image:

```
git clone https://github.com/erigontech/diagnostics.git
cd diagnostics

# Note: https://github.com/erigontech/diagnostics/pull/99 is needed to fix the build
make build-docker
```

Then start the diagnostics container:

```
docker compose up -d erigon-diagnostics
```

Now you can see the diagnostics page at `http://<your-server-ip>:8080`

Follow the instructions [here](https://erigon.gitbook.io/erigon/diagnostic-tool/setup#id-3.-create-a-new-session-in-the-diagnostic-tool) to create a new session and copy the session pin

Then set `DIAGNOSTICS_SESSION` in `.env`

Now start the erigon-support container to connect the diagnostics session to the erigon node:

```
docker compose up -d erigon-support
```

Once the diagnostics tool is successfully connected to the Erigon node, return to your web browser and reload the page. 
