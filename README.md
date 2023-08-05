A collection of blockchain stress-tests based on [stressoor.js](https://github.com/latticexyz/stressoor.js).

# Quickstart

`sendEth` spams the target RPC with simple ETH transfer transactions at a certain rate.

## Setup

`yarn install`

## Run locally

```bash
yarn start sendEth \
    <number of transactions to send> \
    --http <json rpc url> \ # use --ws for websocket
    --pKey <faucet private key>
```

This will send ETH transfer transactions from your machine to the RPC.

## Run on k8s

`yarn start:k8s <NAME>`

This will spin up a job in the cluster and run the same stress-test with the parameters specified in `infra/.env`. You need to have `kubectl` set up (see [run.sh](/infra/cmd/run.sh)).

## Run other stress-tests

`hash` spams the target RPC with function calls to [Hasher.sol](/contracts/src/Hasher.sol) that compute hashes on-chain.

```bash
# Deploy Hasher.sol
yarn deploy:chain <json rpc url> <private key>
# Do: Set the HASHER_ADDRESS constant in src/hash.ts to the deployed address
yarn start hash
    <number of transactions to send> \
    --http <json rpc url> \
    --pKey <faucet private key>
```

# Examples

## Custom Optimism chain
```
yarn start sendEth 1000 --http https://l2-civic-ivory-ferret-s24oysp9jl.t.conduit.xyz --pKey <faucet private key> --chainId 999999999 -g 2400000000 -n 50 -t 10000

```

## Triton testnet
```
yarn start sendEthNaut 1000 --http https://triton.api.nautchain.xyz --pKey <faicet private key> --chainId 91002 -g 2400000000 -n 50 -t 1000000
```

# Benchmarking

## Which metrics are considered
Time, latency, block numbers.

## Time
Time in milliseconds from when the stress test begins and ends. If 1 txn is sent, the time =/= the latency of that txn, as the latency is measured separately. This is the best metric for measuring how long it actually takes to send the number of txns you wish to send, and received all the finalized txns.

There is some extra time taken to retrieve the txn receipt in this time, so it may not be the most accurate for tps measurement.

## Transaction hashes
Instead of using the time in milliseconds for tps, one interesting way of measuring tps is to consider block time and the amount of txns you can fit in a block.

To measure tps with the above method, run a stress test with a max tps value higher than the tps you anticipate, and a max number of addresses that is not too low (around 50 for 1k txns is a good, and any higher probably will not benefit the result that much). After the stress tests have been run, find the block number including the txns you've sent, and divide the number of txns in that block with the block time. 

You will get the block numbers through the stress test, or you can choose one txn hash from the logged txn hashes from the stress test, and find which block the txns have been included in to look at the block on the block explorer. I would recommend searching for the block on the block explorer to get the exact number of txns included in the block.

Goerli optimism block explorer: https://explorerl2new-civic-ivory-ferret-s24oysp9jl.t.conduit.xyz

# Running stress-tests

Stress-test parameters can be specified in `src/config.ts` and in the command-line (non-exhaustive). The latter takes preference.

```
> yarn start --help

Usage: chain-stress-test [options] <test> <nTx>

run a stressoor.js stress test on a chain

Arguments:
  test                     name of the test to run e.g., sendEth
  nTx                      number of transactions

Options:
  -w, --ws <string>        websocket rpc url
  -h, --http <string>      json rpc url
  -c, --chainId <number>   chain ID
  -k, --pKey <string>      faucet private key
  -g, --gasPrice <number>  gas price
  -n, --nAddr <number>     max number of addresses
  -t, --tps <number>       max tps
  -s, --seed <number>      seed
  --help                   display help for command
```

# Making your own stress-test

## Transfers/contract calls

For simple transfer/contract call transactions, looking through [sendEth.ts](/src/sendEth.ts) or [hash.ts](/src/hash.ts) (respectively) and modifying a copy should be quite simple.

Remember to add the test to `index.ts` to be able to call it with `yarn start`.

## Other stress-tests

For complex stress-tests you might want to familiarize yourself with the fundamentals of stressoor.js and create a custom version of [stdtest.ts](/src/stdtest.ts).
This also applies to stress-tests that do things other than send transactions e.g. read-only RPC calls.

## Kubernetes

When running tests from a k8s cluster, the local `/src` directory gets mounted into the pod. This way you can work on a test and quickly run it on the cluster without having to build a docker image. This is currently done by creating a configmap from all the files in the directory (effective but limited).

## Reports

To customize what data get's reported, look into [reportStack.ts](/src/reportStack.ts).
