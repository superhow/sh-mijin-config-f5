# NEM Catapult v.F3 Server Config Scripts by Super How?

##### Table of Contents

[Introduction](#Introduction)

[How-To](#How-To)

[Concepts](#Concepts)

<hr>

## Introduction

_NOTE: These scripts are for configuring a Catapult node after it has already been compiled._

Most of the scripts were originally created by CrackTheCode016 and by mijin core developer Jaguar0625.

**Super How?** adapted to our needs, fixed some issues, deleted unused stuf, addes a few extra features, as well as some additional documentation. Use these scripts as is, without any liabilities to:

- Reset your Catapult node.
- Configure your node for `peer`, `api` or `dual` mode with the necessary extensions for each mode.
- Configure `resources` with a user-given keypair and necessary catapult-server paths.
- Generate a nemesis block properties file with boot keys and paths.
- Generate mosaic ids for the given public key.
- Generate a list of harvester and currency keys and addresses, and configure the nemesis block file with them.
- Generate a unique generation hash for each instance of the network.
- Generate the nemesis block, 64 byte hashes.dat, and a fresh `data` directory.

This is the setup that is required to get a Catapult node up and running. You can change scripts to your needs.

## How-To

### Directory Structure

In order for these to work, a certain directory structure has to be adopted:

```
catapult-data/
├── data
├── nemesis
├── resources
├── scripts
├── seed
└── tmp  
```

Node configuration scripts are inside of the `scripts/` directory.

Firstly, create a directory to house the above structure:

```
mkdir catapult-data && cd catapult-data
mkdir data
mkdir nemesis
mkdir resources
mkdir scripts
mkdir seed
```

Now, go ahead and move the `cat-config` folder over to `catapult-data/scripts`

If you have a remote mongoDB host then you would also want to set the `REMOTE_MONGODB_HOST` global environment variable.

```
export REMOTE_MONGODB_HOST=1.1.1.1:27018

```

### Running `reset.sh`

`reset.sh` will reset any existing node configuration and configure a new node. Keep in mind the scripts utilize the `zsh` shell. Make sure you are in `catapult-data/` when you run this:

`zsh scripts/cat-config/reset.sh --<node_option> <node_type> <path_to_catapult_bin> <private_key> <public_key>`

Let's break this down:

- `<node_type>` - This argument tells the script which kind of node to configure. 
There are three node types in catapult: `dual`, `peer`, and `api`. An explanation for each can be found in #concepts.

- `<path_to_catapult_bin>` - The FULL path to your catapult-server directory on your machine.
Path to where built binaries are i.e. `~/catapult/`.

- `<private_key>` - Server node's new private key or Nemesis new private key in case of new chain.

- `<public_key>` - Existing network public key or Nemesis new public key in case of new chain.

- `<node_option>` - This argument tells the script to configure the node as a new chain node. 
Will connect your node to an existing chain, or join the Foundation network:

- --local - ```zsh scripts/cat-config/reset.sh --local <node_type> <path_to_catapult_bin> <private_key> <public_key>```. This starts a new chain in independent local node. It has its own new generation hash.

- --foundation (in progress)

- --existing - ```zsh scripts/cat-config/reset.sh --existing <node_type> <path_to_catapult_bin> <private_key> <network_public_key> <template_name>```.  
Resources are loaded from template to join an existing network. You may add your own template by copying the structure in `templates/testnet`.

If all went well, you should see the nemesis block information at the bottom of the output. At any time if you want to change your node configuration, you may run `reset.sh` with different settings. 
**Keep in mind that it will delete and reset your chain!**

### Configure `config-harvesting.properties`

Before starting your node, take a private key from `harvester_addresses.txt`, generated during `reset.sh` and go into `resources/config-harvesting.properties`. Set the `harvestKey` to the private key you copied from `harvester_addresses.txt`.

### Starting Catapult with `start.sh`

To start a Catapult service (`server` or `broker`), run this script from `scripts/cat-config`:

`zsh scripts/cat-config/start.sh <server | broker> <path_to_catapult-server_src> --force`

Let's break this down:

- `server | broker` - This argument allows you to pass in the type of service you would like to start. `server` starts up your actual node, while `broker` will start Catapult's broker service.

  - Starting the server: `zsh scripts/cat-config/start.sh server <path_to_catapult-server_src> --force`

  - Starting the broker: `zsh scripts/cat-config/start.sh broker <path_to_catapult-server_src> --force`

- `--force` - This optional argument deletes the `.lock` file that is usually generated in `data/`.

- `--lldb` - This optional argument puts you into debuging mode using LLDB.

## Concepts

### Catapult Extensions

Extensions in Catapult are essentially modules that add features needed by `catapult-server` and are critical for consensus and networking.

### Catapult Node Types

These scripts introduce several interesting concepts about Catapult nodes:

- `peer` nodes are the "bare minimum" version of a Catapult node. It does not load API extensions, but runs the backbone of the `catapult-server` architecture that handles consensus and networking.

- `api` nodes introduce APIs to interface with a given peer node. They load extensions to enable the API node to identify and store partial aggregate bonded transactions.

- `dual` nodes combine a `peer` node and `api` node into a single server instance. It loads extensions required for both `peer` and `api` nodes. 

###
