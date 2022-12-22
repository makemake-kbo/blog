---
title: How to set up a node in a post-merge Ethereum
date: 2022-11-01
---

Setting up an Ethereum node can be quite daunting, even more so in a post-merge world. The most common pitfalls come from having to run 2 clients and having to set up JWT authentication so the clients can talk to each other and know the data isn't getting manipulated. For this demo, we will be using Erigon and Prysm and we will be syncing the Sepolia testnet. 

## Introduction

While we will be syncing sepolia, the process for setting up different combinations of clients, and different networks, should be more or less identical; With the only real differences being the installation of the clients and different names of command line arguments. Most clients offer configuration via a YAML or a TOML file, but for the sake of simplicity, we will stick to only using the command line for setting up our clients. 

## Step 1: Creating a JWT token

Ever since the Ethereum merge node operators are required to run 2 clients. One for the execution layer (processing of transactions, storing balances and monkey jpegs) and one for the consensus layer (producing and validating blocks, gossiping transactions, and blocks). For these clients to communicate with each other we either use an IPC or an HTTP server. Some clients do not have support for IPC so we will only cover setting up a connection between the two via HTTP. The communication API for this is called the Engine API.

To communicate securely over HTTP, the CL and EL need to have the same JWT token. A JWT token is just a simple file containing some bytes that the clients can use to authenticate each other. **We can generate it via OpenSSL like this:**

```bash
openssl rand -hex 32 | tr -d "\n" > "$HOME/jwtsecret"
```

The `"$HOME/jwtsecret"` part of the command above will be the path of our JWT token. Feel free to set this path to wherever you want your token to be.

## Step 2: Setting up the Execution layer

The execution layer is used to process transactions and store data related to the chain(ie. state). Before the merge, node operators only needed to run an EL client to participate in the network, but since the CL is now handling the P2P communication, node operators now need to run both. We will be using Erigon as our EL. First, we need to get erigon on our machine.    

**ead on over to the _Erigon github_, and either clone the repository and compile it yourself(harder) or download a release from the releases tab.** I recommend compiling from source, but downloading a pre-built binary is fine as well. **For this tutorial, we are assuming that the EL binary is located in the same directory and that we have a terminal shell inside of that directory.**

After we got Erigon, we're going to run it with this command:

```bash
./erigon --chain=sepolia --prune=hrtc --datadir="./erigon-sepolia" --ws --authrpc.jwtsecret="$HOME/jwtsecret" --authrpc.port=8551 --authrpc.addr 127.0.0.1 --http --http.api=engine
```

Let's explain what each of these options does one by one: 
- `--chain=sepolia` - This sets the chain that we want to sync. In this case, we want to sync the Sepolia testnet. 
- `--prune=hrtc` - This sets the pruning mode to "hrtc". This means that Erigon will only store the data that is required for the execution layer. For more information on pruning modes, see the Erigon docs.
- `--datadir="./erigon-sepolia"` - This sets the path for the Erigon database. This is the location where Erigon will store the state. 
- `--ws` - This enables the WebSockets API. The WebSockets API is used to communicate with the consensus layer. 
- `--authrpc.jwtsecret="$HOME/jwtsecret" --authrpc.port=8551 --authrpc.addr 127.0.0.1` - feeds the JWT token to the client, sets the port and address the Engine API will listen to
- `--http --http.api=engine` - This enables the HTTP API and sets the API to only expose the Engine API. 

## Step 3: Setting up the Consensus layer

Consensus layer clients are a relatively new addition to the ethereum family. Post-merge we need them to stay in sync with the network, and for some users to participate in the consensus by validating and producing blocks. For our consensus client, i've picked Prysm and we will not be going over how to set up a validator.   

First, open a new terminal shell, preferably inside of the folder where the EL binary is located, **and execute the following commands**:

```bash
mkdir prysm && cd prysm
```
```bash
curl https://github.com/eth-clients/merge-testnets/blob/main/sepolia/genesis.ssz
```
```bash
curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.sh --output prysm.sh && chmod +x prysm.sh
```
After the script has finished running, **we can start our beacon chain node, for this run this command:**

```bash
./prysm.sh beacon-chain --execution-endpoint=http://localhost:8551 --sepolia --jwt-secret=\$HOME/jwtsecret --genesis-state=./genesis.ssz --bootstrap-node="enr:-Iq4QMCTfIMXnow27baRUb35Q8iiFHSIDBJh6hQM5Axohhf4b6Kr_cOCu0htQ5WvVqKvFgY28893DHAg8gnBAXsAVqmGAX53x8JggmlkgnY0gmlwhLKAlv6Jc2VjcDI1NmsxoQK6S-Cii_KmfFdUJL2TANL3ksaKUnNXvTCv1tLwXs0QgIN1ZHCCIyk,enr:-KG4QE5OIg5ThTjkzrlVF32WT_-XT14WeJtIz2zoTqLLjQhYAmJlnk4ItSoH41_2x0RX0wTFIe5GgjRzU2u7Q1fN4vADhGV0aDKQqP7o7pAAAHAyAAAAAAAAAIJpZIJ2NIJpcISlFsStiXNlY3AyNTZrMaEC-Rrd_bBZwhKpXzFCrStKp1q_HmGOewxY3KwM8ofAj_ODdGNwgiMog3VkcIIjKA,enr:-KG4QMJSJ7DHk6v2p-W8zQ3Xv7FfssZ_1E3p2eY6kN13staMObUonAurqyWhODoeY6edXtV8e9eL9RnhgZ9va2SMDRQMhGV0aDKQS-iVMYAAAHD0AQAAAAAAAIJpZIJ2NIJpcIQDhAAhiXNlY3AyNTZrMaEDXBVUZhhmdy1MYor1eGdRJ4vHYghFKDgjyHgt6sJ-IlCDdGNwgiMog3VkcIIjKA"
```
Let's explain what each of these options does one by one:
- `--execution-endpoint=http://localhost:8551` - This sets the endpoint for the execution layer. 
- `--sepolia` - This sets the chain that we want to sync. In this case, we want to sync the Sepolia testnet. 
- `--jwt-secret=\$HOME/jwtsecret` - This sets the path to our JWT token that we generated in step 1. 
- `--genesis-state=./genesis.ssz` - This sets the path to the genesis state that we downloaded in step 3. ***This is only necessary for sepolia*** and new ephemeral testnets that don't come with genesis blocks by default. 
- `--bootstrap-node` - This sets the bootstrap node that we want to connect to. Useful for getting peers. Keep in mind that the ***bootnodes provided are only for sepolia.***

## Conclusion

And that's it! You have now set up a consensus layer client and an execution layer client. You can now participate in a post-merge Ethereum network.

## Useful resources

- [Erigon docs](https://github.com/ledgerwatch/erigon/tree/devel/docs)
- [Prysm docs](https://docs.prylabs.network/docs/getting-started)