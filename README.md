# node-launch

```
git clone https://github.com/Snapchain/node-launch.git
cd node-launch
```

## Launch L1 node

Launch [Erigon](https://github.com/ledgerwatch/erigon) and [Prysm](https://github.com/prysmaticlabs/prysm) to provide L1 RPC of Testnet [Sepolia](https://sepolia.etherscan.io/).

### Generate JWT token

The HTTP connection between the beacon node and execution node needs to be authenticated using a JWT token. Refer to the Prysm [authentication](https://docs.prylabs.network/docs/execution-node/authentication) for more details.

Use OpenSSL to create the token via command:

```
cd sepolia
openssl rand -hex 32 | tr -d "\n" > "jwt.hex"
```

Launch L1 node:

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