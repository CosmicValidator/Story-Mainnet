# Story-Mainnet

## Mainnet Resources

* **Cosmos RPC**: [https://story-rpc.cosmicvalidator.com](https://story-rpc.cosmicvalidator.com)
* **Cosmos API**: [https://story-api.cosmicvalidator.com](https://story-api.cosmicvalidator.com)
* **EVM RPC**: [https://story-evm.cosmicvalidator.com](https://story-evm.cosmicvalidator.com)
* **EVM WSS**: [wss://story-evm-wss.cosmicvalidator.com](wss://story-evm-wss.cosmicvalidator.com)
* **Snapshots**: [https://story-mainnet.cosmicvalidator.com](https://story-mainnet.cosmicvalidator.com)

---

## Installation & Setup

Follow any one of the external guides for full node setup:

* [Full Node Setup](https://docs.story.foundation/network/participate/validators/node-setup-mainnet)

---
## Genesis
* [genesis.json](https://story-mainnet.cosmicvalidator.com/story/mainnet/genesis.json)
```bash
curl -Ls https://story-mainnet.cosmicvalidator.com/story/mainnet/genesis.json > $HOME/.story/story/genesis.json
```
      
## Addrbook 
* [addrbook.json](https://story-mainnet.cosmicvalidator.com/story/mainnet/addrbook.json)
```bash
curl -Ls https://story-mainnet.cosmicvalidator.com/story/mainnet/addrbook.json > $HOME/.story/story/addrbook.json
```

## Snapshot Service

The CosmicValidator snapshot service provides fast synchronization for both the consensus and execution layers using ZSTD compression. Snapshot details are updated every 6 hours.

Snapshot metadata API: [https://story-mainnet.cosmicvalidator.com/api/info](https://story-mainnet.cosmicvalidator.com/api/info)

### Snapshot Download & Extraction Steps

1. Install required dependencies:

```bash
sudo apt install curl jq tar zstd aria2
```

2. Define the data directory path:

```bash
export STORY_HOME="$HOME/.story"
```

3. Fetch snapshot metadata and extract filenames:

```bash
SNAPSHOT_META=$(curl -s https://story-mainnet.cosmicvalidator.com/api/info)
CONSENSUS_FILE=$(echo "$SNAPSHOT_META" | jq -r '.snapshots.consensus')
EXECUTION_FILE=$(echo "$SNAPSHOT_META" | jq -r '.snapshots.execution')
```

4. Download snapshots using the filenames:

```bash
aria2c -x 8 -s 8 https://story-mainnet.cosmicvalidator.com/story/mainnet/$CONSENSUS_FILE
aria2c -x 8 -s 8 https://story-mainnet.cosmicvalidator.com/story/mainnet/$EXECUTION_FILE
```

5. Extract the archives:

```bash
tar -I zstd -xf $CONSENSUS_FILE -C "$STORY_HOME/story"
tar -I zstd -xf $EXECUTION_FILE -C "$STORY_HOME/geth/story/geth"
```

6. (Optional) Clean up the snapshot files:

```bash
rm $CONSENSUS_FILE $EXECUTION_FILE
```

---

## State Sync

State sync allows a node to quickly join the network by syncing state data up to a recent height instead of replaying all blocks.

Make sure to fetch the latest trust height and trust hash before enabling state sync. Use an RPC like:

```bash
curl -s https://story-rpc.cosmicvalidator.com/block | jq .result.block.header
```

Example `config.toml` changes:

```toml
[statesync]
enable = true
rpc_servers = "https://story-rpc.cosmicvalidator.com,https://story-rpc.cosmicvalidator.com"
trust_height = <LATEST_BLOCK_HEIGHT>
trust_hash = "<LATEST_BLOCK_HASH>"
trust_period = "168h0m0s"
```

Restart the node to start state sync.
