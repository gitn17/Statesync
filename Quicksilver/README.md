## Install dependencies if needed
```bash
sudo apt install -y curl jq
```
## Stop the service
```bash
sudo systemctl stop quicksilverd
```
## Backup priv_validator_state.json
```bash
cp $HOME/.quicksilverd/data/priv_validator_state.json $HOME/.quicksilverd/priv_validator_state.json.bak
```
## Reset database
```bash
quicksilverd tendermint unsafe-reset-all --home $HOME/.quicksilverd --keep-addr-book
```
## Fetch up to date statesync configuration
```bash
RPC="https://quicksilver.rpc.ghostinnet.com:443"
LATEST_HEIGHT=$(curl -s $RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.quicksilverd/config/config.toml
```
## Restore priv_validator_state.json
```bash
mv $HOME/.quicksilverd/priv_validator_state.json.bak $HOME/.quicksilverd/data/priv_validator_state.json
```
## Restart the service
```bash
sudo systemctl restart quicksilverd && sudo journalctl -u quicksilverd -f
```
## Disable statesync after synchronization
```bash
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1false|" $HOME/.quicksilverd/config/config.toml
```
