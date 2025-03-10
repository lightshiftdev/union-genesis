# Requirements
The requirements for mainnet node operation are the same as the [Union Testnet node requirements](https://docs.union.build/infrastructure/node-operators/requirements/). In summary, this means:
## Software
- Linux based operating system
## Hardware
- 16 Cores
- 64 GB RAM
- 500 GB Storage
- x86_64 or aarch64 architecture

# Running the Node
There are two primary methods of running the node: the `uniond` binary or Unionvisor. The `uniond` binary is the Cosmos SDK based binary with statically linked libraries. `uniond` is the most flexible option. Unionvisor is our in-house equivalent to Cosmovisor. Unionvisor may not be as flexible as the raw binary, but has support for nix flake configurations - enabling reliable deterministic deployments. Unionvisor is also released as a docker image which bundles all versions of the binary needed to join the network from genesis.

You're welcome to try and run `uniond` with Cosmovisor, but we will not supply official support for this.
## Uniond
You will need to ensure the account running the `uniond` binary has access to it's home folder. By default, this will be located at `~/.union`. We recommend always providing the `--home` flag when starting uniond or issuing any sub-commands. To simplify this, you can create a shell alias to `uniond --home $UNION_HOME`.

### Initialization

You will need to use `uniond` to initialize your node in your desired home directory with the `uniond init` sub-command. This command requires you to supply your moniker. It's recommended to also supply the `--home` and `--chain-id` while running this command. Here is an example

```sh
uniond init poisonphang --chain-id union-1 --home ~/.union
```

#### Notable Files
Once initialized, your node home will contain several `toml` and `json` files which will determine how `uniond` is run. At this point, you should ensure that `config/node_key.json` and `config/priv_validator_key.json` are **the same** as the ones you used to create your genesis transaction. You should also replace the default genesis file with the one from [unionlabs/genesis](https://github.com/unionlabs/genesis)

:::note
The please wait to replace the genesis until it's been finalized.
:::

```
../.union
├── config
│   ├── app.toml
│   ├── client.toml
│   ├── config.toml
│   ├── genesis.json
│   ├── node_key.json
│   └── priv_validator_key.json
└── data
    └── priv_validator_state.json
```

## Unionvisor
Unionvisor can be run and configured as a binary, a Docker image, or via a nix configuration. The most important step with any of these options is changing our your `config/node_key.json` and `config/priv_validator_key.json` and `config/genesis.json`. If you're using nix, you can deploy these via your keys via the nix configuration. You can use something [Mic92/sops-nix](https://github.com/Mic92/sops-nix) for secret management if you're storing your configuration in a public way.

When calling Unionvisor, the following flags will need to be set:
- `--root` or `UNIONVISOR_ROOT`: the home directory of unionvisor (usually one directory above the `uniond` home)
- `--bundle` or `UNIONVISOR_BUNDLE`: the path to the network bundle for Unionvisor 
### Binary
You will need to either always supply all flags or ensure environment variables Unionvisor reads are always set. You will need to manually construct the bundle folder.
### Docker Image
The Docker images come packaged with the bundle and will set environment variables for you (within the container).
### Nix
With nix, you'll want to configure each of the flags in you config before deploying, nix will then handle setting those values.

## Configuration
At this point, there are a few configuration items within your `toml` files that you should set before starting your node for the first time. This guide will supply you with required configurations, but feel free to otherwise configure the node to your liking.

### Client Configuration
Within `config/client.toml` the following configuration options should be set:

#### Chain ID
```toml
# The network chain ID
chain-id = "union-1"
```

### App Configuration
Within `config/app.toml` the following configuration options should be set:
#### Minimum Gas Price
```toml
# The minimum gas prices a validator is willing to accept for processing a
# transaction. A transaction's fees must meet the minimum of any denomination
# specified in this config (e.g. 0.25token1;0.0001token2).
minimum-gas-prices = "1ugas"
```

### Node Configuration
Within `config/config.toml` the following configuration options should be set:
#### Consensus
```toml
# How long we wait for a proposal block before pre-voting nil
timeout_propose = "3s"
# How much timeout_propose increases with each round
timeout_propose_delta = "500ms"
# How long we wait after receiving +2/3 prevotes for “anything” (ie. not a single block or nil)
timeout_prevote = "1s"
# How much the timeout_prevote increases with each round
timeout_prevote_delta = "500ms"
# How long we wait after receiving +2/3 precommits for “anything” (ie. not a single block or nil)
timeout_precommit = "1s"
# How much the timeout_precommit increases with each round
timeout_precommit_delta = "500ms"
# How long we wait after committing a block, before starting on the new
# height (this gives us a chance to receive some more precommits, even
# though we already have +2/3).
timeout_commit = "5s"
```
#### Seeds
*awaiting Lavender.Five's seed nodes*
```toml
seeds = ""
```
#### Seed Mode
Validator nodes and seed nodes should be exclusive from one another. If you're configuring a validator, please ensure `seed_mode` is set to false.
```toml
# Seed mode, in which node constantly crawls the network and looks for
# peers. If another node asks it for addresses, it responds and disconnects.
#
# Does not work if the peer-exchange reactor is disabled.
seed_mode = false
```
