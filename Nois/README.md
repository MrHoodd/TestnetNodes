# Official Links
[Website](https://nois.network/) [Twitter](https://twitter.com/NoisNetwork) [Discord](https://discord.gg/JvUrJJeGXn)

# Explorer
[Explorer](https://explorer.kjnodes.com/nois/staking)

# Install Node Guide

### Setting up variables
Specify the name of your moniker (validator) which will be visible in the explorer
```bash
NODENAME=<YOUR_MONIKER_NAME>
```
### Save and import variables into system
```bash
NOIS_PORT=30
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export NOIS_CHAIN_ID=nois-testnet-003" >> $HOME/.bash_profile
echo "export NOIS_PORT=${NOIS_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Update Packages and Depencies
```bash
sudo apt update && sudo apt upgrade -y
```

### Install Depencies
```bash
sudo apt install curl build-essential git wget jq make gcc tmux chrony -y
```

### Install GO
```bash
ver="1.18.2" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

### Download binaries
```bash
cd $HOME
git clone https://github.com/noislabs/full-node.git 
cd full-node/full-node/
./build.sh
mv out/noisd $HOME/go/bin/
```

### Config app
```bash
noisd config chain-id $NOIS_CHAIN_ID
noisd config keyring-backend test
noisd config node tcp://localhost:${NOIS_PORT}657
```

### Init app
```bash
noisd init $NODENAME --chain-id $NOIS_CHAIN_ID
```

### Download genesis and addrbook
```bash
wget -qO $HOME/.noisd/config/genesis.json "https://raw.githubusercontent.com/noislabs/networks/main/nois-testnet-003/genesis.json"
wget -O $HOME/.noisd/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Nois/addrbook.json"
```
### Set seeds, and peers, minimum gas price
```bash
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.noisd/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.noisd/config/config.toml
peers="2bf8002d0f65c3d86fca31ea0f043d912682c3e0@65.109.70.23:17356"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.noisd/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.noisd/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.noisd/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.noisd/config/config.toml
export DENOM=unois
export CONFIG_DIR=$HOME/.noisd/config
sed -i 's/minimum-gas-prices = ""/minimum-gas-prices = "0.05'"${DENOM}"'"/' $CONFIG_DIR/app.toml \
  && sed -i 's/^timeout_propose =.*$/timeout_propose = "2s"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_propose_delta =.*$/timeout_propose_delta = "500ms"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_prevote =.*$/timeout_prevote = "1s"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_prevote_delta =.*$/timeout_prevote_delta = "500ms"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_precommit =.*$/timeout_precommit = "1s"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_precommit_delta =.*$/timeout_precommit_delta = "500ms"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_commit =.*$/timeout_commit = "2s"/' $CONFIG_DIR/config.toml
  ```

### Set custom ports
```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${NOIS_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${NOIS_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${NOIS_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${NOIS_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${NOIS_PORT}660\"%" $HOME/.noisd/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${NOIS_PORT}317\"%; s%^address = \":8080\"%address = \":${NOIS_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${NOIS_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${NOIS_PORT}091\"%" $HOME/.noisd/config/app.toml
```

### Config pruning (Optional)
```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.noisd/config/app.toml
```

### Indexer (Optional)
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.noisd/config/config.toml
```

### Enable prometheus
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.noisd/config/config.toml
```

### Reset chain data
```bash
noisd tendermint unsafe-reset-all --home $HOME/.noisd
```

### Create service
```bash
sudo tee /etc/systemd/system/noisd.service > /dev/null <<EOF
[Unit]
Description=nois
After=network-online.target

[Service]
User=$USER
ExecStart=$(which noisd) start --home $HOME/.noisd
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### Register and start service
```bash
sudo systemctl daemon-reload
sudo systemctl enable noisd
sudo systemctl restart noisd && sudo journalctl -u noisd -f -o cat
```

### State-Sync (Optional)
```bash
peers="8073bd66d5fa581c7b3d0a08d0df1fe318d70d99@135.181.35.46:55656"; \
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.noisd/config/config.toml

SNAP_RPC="http://135.181.35.46:55657"; \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash); \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"|" ~/.noisd/config/config.toml

sudo systemctl stop noisd && \
noisd tendermint unsafe-reset-all --home $HOME/.noisd && \
sudo systemctl restart noisd
```

### Create wallet
To create a new wallet, don't forget to save the mnemonics 
```bash
noisd keys add $WALLET
```

To recover existing keys use
```bash
noisd keys add $WALLET --recover
```

List of wallets
```bash
noisd keys list
```
### Save wallet info
Add wallet and valoper address into variables 
```bash
NOIS_WALLET_ADDRESS=$(noisd keys show $WALLET -a)
NOIS_VALOPER_ADDRESS=$(noisd keys show $WALLET --bech val -a)
echo 'export NOIS_WALLET_ADDRESS='${NOIS_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export NOIS_VALOPER_ADDRESS='${NOIS_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Ask for faucet
```bash
curl --header "Content-Type: application/json" \
--request POST \
--data '{"denom":"unois","address":"'$NOIS_WALLET_ADDRESS'"}' \
http://faucet.noislabs.com/credit
```

### To check your wallet balance:
```bash
noisd query bank balances $NOIS_WALLET_ADDRESS
```

### Create validator
After your node is synced, create validator

To check if your node is synced simply run
```bash
noisd status 2>&1 | jq .SyncInfo
```
To create your validator run command below
```bash
noisd tx staking create-validator \
  --amount 100000000unois \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(noisd tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $NOIS_CHAIN_ID
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --fees=10000unois
```
 
## Usefull commands
### Service management
Check logs
```bash
journalctl -fu noisd -o cat
```

Start service
```bash
sudo systemctl start noisd
```

Stop service
```bash
sudo systemctl stop noisd
```

Restart service
```bash
sudo systemctl restart noisd
```

### Node info
Synchronization info
```bash
noisd status 2>&1 | jq .SyncInfo
```

Validator info
```bash
noisd status 2>&1 | jq .ValidatorInfo
```

Node info
```bash
noisd status 2>&1 | jq .NodeInfo
```

Show node id
```bash
noisd tendermint show-node-id
```

### Wallet operations
List of wallets
```bash
noisd keys list
```

Recover wallet
```bash
noisd keys add $WALLET --recover
```

Delete wallet
```bash
noisd keys delete $WALLET
```

Get wallet balance
```bash
noisd query bank balances $NOIS_WALLET_ADDRESS
```

Transfer funds
```bash
noisd tx bank send $NOIS_WALLET_ADDRESS <TO_NOIS_WALLET_ADDRESS> 10000000unois
```

### Voting
```bash
noisd tx gov vote 1 yes --from $WALLET --chain-id=$NOIS_CHAIN_ID
```

### Staking, Delegation and Rewards
Delegate stake
```bash
noisd tx staking delegate $NOIS_VALOPER_ADDRESS 10000000unois --from=$WALLET --chain-id=$NOIS_CHAIN_ID --fees=10000unois
```

Redelegate stake from validator to another validator
```bash
noisd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000unois --from=$WALLET --chain-id=$NOIS_CHAIN_ID --fees=10000unois
```

Withdraw all rewards
```bash
noisd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$NOIS_CHAIN_ID --fees=10000unois
```

Withdraw rewards with commision
```bash
noisd tx distribution withdraw-rewards $NOIS_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$NOIS_CHAIN_ID --fees=10000unois
```

### Validator management
Edit validator
```bash
noisd tx slashing unjail \
  --moniker=$NODENAME \
  --identity="<your_keybase_id>" \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$NOIS_CHAIN_ID \
  --from=$WALLET \
  --fees=10000unois
```

Unjail validator
```bash
noisd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$NOIS_CHAIN_ID \
  --fees=10000unois
```

Delete node
```bash
sudo systemctl stop noisd
sudo systemctl disable noisd
sudo rm /etc/systemd/system/nois* -rf
sudo rm $(which noisd) -rf
sudo rm $HOME/.noisd* -rf
sudo rm $HOME/nois -rf
sed -i '/NOIS_/d' ~/.bash_profile
```
