# Althea Mainnet state sync guide

1. Stop Althea background service and reset database:
```
systemctl stop althea
althea tendermint unsafe-reset-all --home $HOME/.althea --keep-addr-book
```
2. Use variables to connect to RPC node and set the number and the hash of the block for state sync:
```
PEER="46bbca247297be28bbc318af024147efa012f9a2@194.60.201.146"
SNAP="http://194.60.201.146:45057"
LATEST_HEIGHT=$(curl -s $SNAP/block | jq -r .result.block.header.height)
TRUST_HEIGHT=$((LATEST_HEIGHT - 2000))
TRUST_HASH=$(curl -s "$SNAP/block?height=$TRUST_HEIGHT" | jq -r .result.block_id.hash)

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP,$SNAP\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$TRUST_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" $HOME/.althea/config/config.toml
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEER\"/" $HOME/.althea/config/config.toml
```
3. Run Althea background service:
```
sudo systemctl start althea
sudo journalctl -u althea -f -o cat
```
