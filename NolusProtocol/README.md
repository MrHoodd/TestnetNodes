# Official Links
[Website](https://nolus.io/) [Twitter](https://twitter.com/nolusprotocol) [Discord](https://discord.com/invite/nolus-protocol) [Documents](https://docs-nolus-protocol.notion.site/Run-a-Full-Node-7a92545223e7483bb4a02cce30b53aa8)

## [Explorer](https://nolus.explorers.guru/validators) [Explorer](https://explorer-rila.nolus.io/nolus-rila/staking)

# Install Node Guide Nolus Protocol
### Setting up variables

Specify the name of your moniker (validator) which will be visible in the explorer
```bash
NODENAME=<YOUR_MONIKER_NAME>
```
### Save and import variables into system
```bash
NOLUS_PORT=44
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export NOLUS_CHAIN_ID=nolus-rila" >> $HOME/.bash_profile
echo "export NOLUS_PORT=${NOLUS_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Update Packages and Depencies
```bash
sudo apt update && sudo apt upgrade -y
```

### Install Depencies
```bash
sudo apt install curl build-essential git wget jq make gcc tmux chrony tar htop net-tools clang pkg-config libssl-dev ncdu -y
```

### Install GO
```bash
if ! [ -x "$(command -v go)" ]; then
  ver="1.19.1"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

### Download binaries
```bash
cd $HOME && rm -rf nolus-core
git clone https://github.com/Nolus-Protocol/nolus-core
cd nolus-core
git checkout v0.1.39
make install
```

### Config app
```bash
nolusd config chain-id $NOLUS_CHAIN_ID
nolusd config keyring-backend test
nolusd config node tcp://localhost:${NOLUS_PORT}657
```

## Init app
```bash
nolusd init $NODENAME --chain-id $NOLUS_CHAIN_ID
```

### Download genesis and addrbook
```bash
wget -O $HOME/.nolus/config/genesis.json "https://raw.githubusercontent.com/Nolus-Protocol/nolus-networks/main/testnet/nolus-rila/genesis.json"
wget -O $HOME/.nolus/config/addrbook.json "https://raw.githubusercontent.com/elangrr/testnet_guide/main/nolus/addrbook.json"
```

## Set seeds and peers
```bash
PEERS="$(curl -s "https://raw.githubusercontent.com/Nolus-Protocol/nolus-networks/main/testnet/nolus-rila/persistent_peers.txt")"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.nolus/config/config.toml
```

### Set minimum gas price
```bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025unls\"/" $HOME/.nolus/config/app.toml
```

## Set custom ports
```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${NOLUS_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${NOLUS_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${NOLUS_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${NOLUS_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${NOLUS_PORT}660\"%" $HOME/.nolus/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${NOLUS_PORT}317\"%; s%^address = \":8080\"%address = \":${NOLUS_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${NOLUS_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${NOLUS_PORT}091\"%" $HOME/.nolus/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:2${NOLUS_PORT}7\"%" $HOME/.nolus/config/client.toml
```

### Config pruning (Optional)
```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.nolus/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.nolus/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.nolus/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.nolus/config/app.toml
```

### Indexer (Optional)
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.nolus/config/config.toml
```

### Enable prometheus
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.nolus/config/config.toml
```

### Reset chain data
```bash
nolusd tendermint unsafe-reset-all --home $HOME/.nolus
```

### Create service
```bash
sudo tee /etc/systemd/system/nolusd.service > /dev/null <<EOF
[Unit]
Description=nolus
After=network-online.target

[Service]
User=$USER
ExecStart=$(which nolusd) start --home $HOME/.nolus
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
sudo systemctl enable nolusd
sudo systemctl restart nolusd && sudo journalctl -u nolusd -f -o cat
```

### State-Sync
```bash
cp $HOME/.nolus/data/priv_validator_state.json $HOME/.nolus/priv_validator_state.json.backup
nolusd tendermint unsafe-reset-all --home $HOME/.nolus
STATE_SYNC_RPC=http://rpc.nolus.ppnv.space:34657
STATE_SYNC_PEER=1a0bb6c35e2663202535d4b849ff06250762d299@rpc.nolus.ppnv.space:35656
LATEST_HEIGHT=$(curl -s $STATE_SYNC_RPC/block | jq -r .result.block.header.height)
SYNC_BLOCK_HEIGHT=$(($LATEST_HEIGHT - 1000))
SYNC_BLOCK_HASH=$(curl -s "$STATE_SYNC_RPC/block?height=$SYNC_BLOCK_HEIGHT" | jq -r .result.block_id.hash)
sed -i.bak -e "s|^enable *=.*|enable = true|" $HOME/.nolus/config/config.toml
sed -i.bak -e "s|^rpc_servers *=.*|rpc_servers = \"$STATE_SYNC_RPC,$STATE_SYNC_RPC\"|" \
  $HOME/.nolus/config/config.toml
sed -i.bak -e "s|^trust_height *=.*|trust_height = $SYNC_BLOCK_HEIGHT|" \
  $HOME/.nolus/config/config.toml
sed -i.bak -e "s|^trust_hash *=.*|trust_hash = \"$SYNC_BLOCK_HASH\"|" \
  $HOME/.nolus/config/config.toml
sed -i.bak -e "s|^persistent_peers *=.*|persistent_peers = \"$STATE_SYNC_PEER\"|" \
  $HOME/.nolus/config/config.toml
mv $HOME/.nolus/priv_validator_state.json.backup $HOME/.nolus/data/priv_validator_state.json
sudo systemctl restart nolusd && journalctl -u nolusd -f --no-hostname -o cat
```

### Create wallet
To create a new wallet, don't forget to save the mnemonics 
```bash
nolusd keys add $WALLET
```

To recover existing keys use
```bash
nolusd keys add $WALLET --recover
```

List of wallets
```bash
nolusd keys list
```
### Save wallet info
Add wallet and valoper address into variables
```bash
NOLUS_WALLET_ADDRESS=$(nolusd keys show $WALLET -a)
NOLUS_VALOPER_ADDRESS=$(nolusd keys show $WALLET --bech val -a)
echo 'export NOLUS_WALLET_ADDRESS='${NOLUS_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export NOLUS_VALOPER_ADDRESS='${NOLUS_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### To check your wallet balance:
```bash
nolusd query bank balances $NOLUS_WALLET_ADDRESS
```
### Create validator
After your node is synced, create validator
```bash
nolusd tx staking create-validator \
--amount=4990000unls \
--pubkey=$(nolusd tendermint show-validator) \
--moniker=$NODENAME \
--identity="" \
--details="" \
--website="" \
--chain-id=$NOLUS_CHAIN_ID \
--commission-rate=0.1 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=$WALLET \
--fees=500unls -y
```

## Usefull commands
### Service management
Check logs
```bash
journalctl -fu nolusd -o cat
```

Start service
```bash
sudo systemctl start nolusd
```

Stop service
```bash
sudo systemctl stop nolusd
```

Restart service
```bash
sudo systemctl restart nolusd
```

### Node info
Synchronization info
```bash
nolusd status 2>&1 | jq .SyncInfo
```

Validator info
```bash
nolusd status 2>&1 | jq .ValidatorInfo
```

Node info
```bash
nolusd status 2>&1 | jq .NodeInfo
```

Show node id
```bash
nolusd tendermint show-node-id
```

### Wallet operations
List of wallets
```bash
nolusd keys list
```

Recover wallet
```bash
nolusd keys add $WALLET --recover
```

Delete wallet
```bash
nolusd keys delete $WALLET
```

Get wallet balance
```bash
nolusd query bank balances $NOLUS_WALLET_ADDRESS
```

Transfer funds
```bash
nolusd tx bank send $NOLUS_WALLET_ADDRESS <TO_OLLO_WALLET_ADDRESS> 10000000unls
```

### Voting
```bash
nolusd tx gov vote 1 yes --from $WALLET --chain-id=$NOLUS_CHAIN_ID --fees=500unls -y
```

### Staking, Delegation and Rewards
Delegate stake
```bash
nolusd tx staking delegate $NOLUS_VALOPER_ADDRESS 10000000unls --from=$WALLET --chain-id=$NOLUS_CHAIN_ID --fees=500unls -y
```

Redelegate stake from validator to another validator
```bash
nolusd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000unls --from=$WALLET --chain-id=$NOLUS_CHAIN_ID --fees=500unls -y
```

Withdraw all rewards
```bash
nolusd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$NOLUS_CHAIN_ID --fees=500unls -y
```

Withdraw rewards with commision
```bash
nolusd tx distribution withdraw-rewards $NOLUS_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$NOLUS_CHAIN_ID --fees=500unls -y
```

### Validator management
Edit validator
```bash
nolusd tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$NOLUS_CHAIN_ID \
  --from=$WALLET \
  --fees=500unls -y
```

Unjail validator
```bash
nolusd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$NOLUS_CHAIN_ID \
  --fees=500unls -y
```

### Delete node
```bash
sudo systemctl stop nolusd && \
sudo systemctl disable nolusd && \
rm /etc/systemd/system/nolusd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .nolus && \
rm -rf $(which nolusd)
sed -i '/NOLUS_/d' ~/.bash_profile
```
