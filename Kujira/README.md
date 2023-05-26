## Install dependencies if needed
```bash
sudo apt install -y curl jq
```
## Stop the service
```bash
sudo systemctl stop kujirad
```
## Reset database
```bash
kujirad tendermint unsafe-reset-all --home $HOME/.kujira --keep-addr-book
```
## Download addrbook if needed
```bash
curl -s https://ghostinnet.com/kujira/addrbook.json > $HOME/.kujira/config/addrbook.json
```
## Fetch up to date statesync configuration
```bash
RPC="https://kujira.rpc.ghostinnet.com:443"
LATEST_HEIGHT=$(curl -s $RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$RPC,$RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.kujira/config/config.toml
```
## Restart the service
```bash
sudo systemctl restart kujirad && sudo journalctl -u kujirad -f
```
## Disable statesync after synchronization
```bash
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1false|" $HOME/.kujira/config/config.toml
```
