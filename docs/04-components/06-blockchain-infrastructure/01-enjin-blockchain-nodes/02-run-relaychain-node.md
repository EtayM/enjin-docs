---
title: "Run a Relaychain Node"
slug: "../run-relaychain-node"
---
There are two different ways in which you can a Node, the first one being through Docker, and the second one through a Binary (either precompiled or [Built from Source](/04-components/06-blockchain-infrastructure/01-enjin-blockchain-nodes/04-building-from-source.md)).

## Create Data Directory

First begin by creating a data directory for the: `sudo mkdir /data`

Then set the permission for it using `sudo chown -R 1000:1000 /data`

## Docker

:::info
The Docker Image can be found on Docker Hub at: [enjin/relaychain](https://hub.docker.com/r/enjin/relaychain).  
In this example, we demonstrate using the version `latest`. However, in practice, we recommend statically setting this to a specific version (such as `v100`) and then performing manual upgrades to your nodes as and when appropriate. This is to ensure that your node doesn't inadvertently differ from one restart to the next. See the [Upgrading using Docker](#upgrading-using-docker) section for more information.

You can use the following `docker-compose.yml` file:

```yaml
services:
  relaychain:
    container_name: relaychain
    image: enjin/relaychain:latest
    ports:
      - 9944:9944
      - 9615:9615
      - 30333:30333
      - 30334:30334
    volumes:
      - /data:/chainstate
    command: [
      "--name=enjin-relay-docker",
      "--rpc-external",
      "--rpc-cors=all",
      "--chain=enjin",
      "--base-path=/chainstate"
    ]
```

Simply run the command `docker-compose up -d` to run the container.

You can use the command `docker ps` to check that the container is running. You can then stream the logs (by taking the `<container_id>`) to check its current sync position by using: `docker logs -f <container_id> --since 1m`

## Binary

### Command

`$ ./enjin --name "enjin-relay-docker" --rpc-external --chain enjin`

Connecting to your node

Depending on the use case, there are a couple of ways to connect to the node:

- WebSocket Connection: `ws://localhost:9944`
- RPC (HTTP) Connection: `http://localhost:9944`

## Archive Node

In order to run an archive node, the following argument needs to be passed to either the binary or added to the command section of the `docker-compose.yml` file:

`--pruning=archive`

This should be appended after line 19 in the `docker-compose.yml` file, or at the end of the Binary execution command.

## Upgrading

:::info It is not always possible to downgrade a node.
In the event the node incurred a database upgrade, it is no longer possible to downgrade the node to an older version without first re-syncing the node.  

Additionally, in the event an on-chain upgrade has been enacted that requires a newer node minimum version, it will not be possible to downgrade below that version as new blocks following the upgrade will fail to import.  
**Always keep your node version up-to-date.**
:::

In order to ensure compatibility with the chain at all times, and to ensure the best security, it is imperative that node operators keep their nodes up-to-date with each release that is published. The process of upgrading is incredibly simple, and in almost all cases, requires very little involvement by the node operator. This is because, the node will automatically detect when it's using an older version of the database and automatically upgrade it to the latest one that is compatible with the 

We recommend that node operators subscribe to our mailing list [mailing-list-node-operators@enjin.io](https://groups.google.com/a/enjin.io/g/mailing-list-node-operators)  in order to be informed when we publish a new node version. However, for those who don't want to subscribe, you can query our [Docker Hub repository](https://hub.docker.com/r/enjin/relaychain/tags) to check for new versions. All versions are automatically pushed to Docker Hub, so you will always be able to find the latest version there.

### Upgrading using Docker

Open your `docker-compose.yml` in a text editor and locate the `image` line, in the below example it is line 4:

```yaml
services:  
  relaychain:  
    container_name: relaychain  
    image: enjin/relaychain:latest  # <-- this line
    ports:  
      - ...
```

If you are using the version `latest` (as in the above example), simply restarting that node at any point will ensure you're automatically updated to the very latest stable version. However, if you have statically set an image version, simply alter the line to specify the latest versioned release that appears on [Docker Hub](https://hub.docker.com/r/enjin/relaychain/tags).

For example, if you are running `enjin/relaychain:v100` and the latest version is `enjin/relaychain:v200` you would simply need to make the following change:

```yaml
services:  
  relaychain:  
    container_name: relaychain  
+    image: enjin/relaychain:v200 # add this line
-    image: enjin/relaychain:v100 # remove this line
    ports:  
      - ...
```

Once updated, simply restart the node by first stopping the existing node and then repeating the steps to start the node using the command `docker-compose up -d` in the directory where the `docker-compose.yml`file is located.

### Upgrading using Binary

Simply download the latest binary and run it as you have always done as per the [Binary, Command](/04-components/06-blockchain-infrastructure/01-enjin-blockchain-nodes/02-run-relaychain-node.md#command) section. It's as simple as that!

#### Upgrading from Source

If you are upgrading based on a build you've produced from the source code, you must navigate to the directory containing the source code in a terminal. Once you have built the binary, simply follow the instruction [Upgrading using Binary](/04-components/06-blockchain-infrastructure/01-enjin-blockchain-nodes/02-run-relaychain-node.md#upgrading-using-binary) to complete the upgrade. 

##### Git-cloned Repository

You should then fetch the latest changes using `git fetch` and then retrieve the source code of the latest tagged version `git checkout <tag>`. Once you have swapped to the latest source code for that release, you can simply run `cargo build --release` (as guided in [Building From Source](/04-components/06-blockchain-infrastructure/01-enjin-blockchain-nodes/04-building-from-source.md)) to produce a new binary using the latest version.

##### Downloaded Repository

In the event that the repository was directly downloaded, and not cloned via `git`, you will first need to download the source code for the latest version. Once done, extract the source code and navigate to the directory in a terminal. You can then run `cargo build --release` (as guided in [Building From Source](/04-components/06-blockchain-infrastructure/01-enjin-blockchain-nodes/04-building-from-source.md)) to produce a new binary.

## Looking to run a validator?

See the [Running a Validator](/04-components/06-blockchain-infrastructure/02-operating-relaychain-validator/01-running-a-validator.md) section.
