# Full Stack Overview

The Cosmos network consists primarily of two classes of users: full nodes and light nodes. 
Full nodes are active participants in the p2p network that verify and execute every block and transaction on the network.
Light nodes are end users that send transactions to, and verify proofs from, the full nodes.

The Full Node software consists of:

- Tendermint - the blockchain engine, handling p2p networking, consensus, and all blockchain data structures
- Gaia - the cryptocurrency transaction and application layer, running atop Tendermint

The Light Node software consists of:

- Cosmos-UI - the user interface for interacting with the blockchain, including the wallet and a governance module
- Light Client Daemon - middleware for connecting the Cosmos-UI to the full nodes

We are interested in bugs in all of these components.
However, some of the software is still in flux, and some of it is only supposed to be used for development.
Therefore, not all packages or repositories are within scope of this Bounty right now.
For an up-to-date description of the scope, please see XXX.

What follows are details on how to get the full stack set up.

## Requirements 

- Install Go, set your `$GOPATH`, and put `$GOPATH/bin` on your `$PATH`
- Install Javascript and `yarn`

## Gaia

The `gaia` tool contains everything required to get started without the UI.

### Install

Install `gaia` as follows:

```
mkdir -p $GOPATH/src/github.com/MrXJC/gaia
git clone https://github.com/MrXJC/gaia $GOPATH/src/github.com/MrXJC/gaia
cd $GOPATH/src/github.com/MrXJC/gaia
make all
```

Note the full paths are important for compiling Go code.
You must also make sure that `$GOPATH/bin` is on your `$PATH`.


### Initialize

To be able to interact with the blockchain we first need a key:

```
gaia client keys new mykey
```

Write down the passphrase to backup the key. Copy the address for later.
You can see the address again with:

```
gaia client keys list
```

### Full Node

Now initialze the blockchain, giving some coins to your address:

```
gaia node init --chain-id local <your address>
```

And start it up:

```
gaia node start
```

Congrats - you're successfully running a one-node blockchain.

Note the `gaia node start` command runs Tendermint under the hood,
and the `gaia node` command shadows much of the `tendermint node` command.
See the [Tendermint docs](https://tendermint.readthedocs.io/en/master/using-tendermint.html#using-tendermint) 
for more info on running nodes and connecting multiple nodes together.

Visit `http://localhost:46657` for a list of available RPC endpoints.
Now we can setup a light node to use this RPC to make transactions
and verify data from the blockchain.

### Light Node

First we need to initialize the light node:

```
gaia client init --chain-id local --node localhost:46657
```

To do this securely on a real public network, we need to obtain a recent validator set digest
and validate it against what's returned. For now, just accept.

Now we can query our account and see our initial balance.

```
gaia client query account <your address>
```

Let's make a new account to send funds to:

```
gaia client keys new another-key
```

Copy the address and send funds:

```
gaia client tx send --to <new address> --amount 10fermion --name mykey
gaia client query account <old address>
gaia client query account <new address>
```

### Cosmos UI

Most users will interface with Cosmos using the Cosmos-UI.
To install the UI:

```
git clone https://github.com/cosmos/cosmos-ui
cd cosmos-ui
yarn install
```

Now to run the UI:

```
COSMOS_NODE=localhost:46657 yarn run testnet local
```

Now within the UI, create a new key. Copy the address,
and use the `gaia client` command to send funds to that key, as before.
Then you will be able to use the UI to make transactions.
