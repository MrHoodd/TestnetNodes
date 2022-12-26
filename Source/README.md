# Official Links
[Website](https://www.sourceprotocol.io/) [Twitter](https://twitter.com/sourceprotocol_) [Discord](https://discord.gg/zj8xxUCeZQ)

# Explorer
[Explorer](https://explorer.kjnodes.com/source-test/staking)

# Install Node Guide sourcechain-testnet
### Setting up variables
Specify the name of your moniker (validator) which will be visible in the explorer
```bash
NODENAME=<YOUR_MONIKER_NAME>
```
### Save and import variables into system
```bash
SOURCE_PORT=28
if [ ! $NODENAME ]; then
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
fi
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export SOURCE_CHAIN_ID=sourcechain-testnet" >> $HOME/.bash_profile
echo "export SOURCE_PORT=${SOURCE_PORT}" >> $HOME/.bash_profile
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
git clone -b testnet https://github.com/Source-Protocol-Cosmos/source.git
cd ~/source
make install
```

### Config app
```bash
sourced config chain-id $SOURCE_CHAIN_ID
sourced config keyring-backend test
sourced config node tcp://localhost:${SOURCE_PORT}657
```

## Init app
```bash
sourced init $NODENAME --chain-id $SOURCE_CHAIN_ID
```

### Download genesis and addrbook
```bash
curl -s  https://raw.githubusercontent.com/Source-Protocol-Cosmos/testnets/master/sourcechain-testnet/genesis.json > ~/.source/config/genesis.json
wget -O $HOME/.source/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Source/addrbook.json"
```

## Set seeds and peers
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0usource\"/;" ~/.source/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.source/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.source/config/config.toml

peers="6ca675f9d949d5c9afc8849adf7b39bc7fccf74f@164.92.98.17:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.source/config/config.toml

seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.source/config/config.toml

sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.source/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.source/config/config.toml
```

## Set custom ports
```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${SOURCE_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${SOURCE_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${SOURCE_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${SOURCE_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${SOURCE_PORT}660\"%" $HOME/.source/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${SOURCE_PORT}317\"%; s%^address = \":8080\"%address = \":${SOURCE_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${SOURCE_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${SOURCE_PORT}091\"%" $HOME/.source/config/app.toml
```

### Config pruning (Optional)
```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.source/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.source/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.source/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.source/config/app.toml
```
### Indexer (optional)
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.source/config/config.toml
```

### Set minimum gas price
```bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0usource\"/" $HOME/.source/config/app.toml
```

### Enable prometheus
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.source/config/config.toml
```

### Reset chain data
```bash
sourced unsafe-reset-all
```

### Create service
```bash
sudo tee /etc/systemd/system/sourced.service > /dev/null <<EOF
[Unit]
Description=source
After=network-online.target

[Service]
User=$USER
ExecStart=$(which sourced) --home $HOME/.source start
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
sudo systemctl enable sourced
sudo systemctl restart sourced && sudo journalctl -u sourced -f -o cat
```
### SnapShot
```bash
cd $HOME
sudo systemctl stop sourced
cp $HOME/.source/data/priv_validator_state.json $HOME/.source/priv_validator_state.json.backup
rm -rf $HOME/.source/data
curl -o - -L http://source.snapshot.stavr.tech:4001/source/source-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.source --strip-components 2
curl -o - -L http://source.wasm.stavr.tech:1050/wasm-snap.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.source/data --strip-components 3
mv $HOME/.source/priv_validator_state.json.backup $HOME/.source/data/priv_validator_state.json
sudo systemctl restart sourced && journalctl -u sourced -f -o cat
```
### Create wallet
To create a new wallet, don't forget to save the mnemonics
```bash
sourced keys add $WALLET
```

To recover existing keys use
```bash
sourced keys add $WALLET --recover
```

List of wallets
```bash
sourced keys list
```

### Save wallet info
Add wallet and valoper address into variables
```bash
SOURCE_WALLET_ADDRESS=$(sourced keys show $WALLET -a)
SOURCE_VALOPER_ADDRESS=$(sourced keys show $WALLET --bech val -a)
echo 'export SOURCE_WALLET_ADDRESS='${SOURCE_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export SOURCE_VALOPER_ADDRESS='${SOURCE_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### To check your wallet balance:
```bash
sourced query bank balances $SOURCE_WALLET_ADDRESS
```

### Create validator
After your node is synced, create validator
```bash
sourced tx staking create-validator \
  --amount 1000000usource \
  --from $WALLET \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(sourced tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $SOURCE_CHAIN_ID
  --fees=100usource -y
```

## Usefull commands
### Service management
Check logs
```bash
journalctl -fu sourced -o cat
```

Start service
```bash
sudo systemctl start sourced
```

Stop service
```bash
sudo systemctl stop sourced
```

Restart service
```bash
sudo systemctl restart sourced
```

### Node info
Synchronization info
```bash
sourced status 2>&1 | jq .SyncInfo
```

Validator info
```bash
sourced status 2>&1 | jq .ValidatorInfo
```

Node info
```bash
sourced status 2>&1 | jq .NodeInfo
```

Show node id
```bash
sourced tendermint show-node-id
```

### Wallet operations
List of wallets
```bash
sourced keys list
```

Recover wallet
```bash
sourced keys add $WALLET --recover
```

Delete wallet
```bash
sourced keys delete $WALLET
```

Get wallet balance
```bash
sourced query bank balances $SOURCE_WALLET_ADDRESS
```

Transfer funds
```bash
sourced tx bank send $SOURCE_WALLET_ADDRESS <TO_SOURCE_WALLET_ADDRESS> 10000000usource --fees=100usource
```

### Voting
```bash
sourced tx gov vote 1 yes --from $WALLET --chain-id=$SOURCE_CHAIN_ID --fees=100usource
```

### Staking, Delegation and Rewards
Delegate stake
```bash
sourced tx staking delegate $SOURCE_VALOPER_ADDRESS 10000000usource --from=$WALLET --chain-id=$SOURCE_CHAIN_ID --fees=100usource
```

Redelegate stake from validator to another validator
```bash
sourced tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000usource --from=$WALLET --chain-id=$SOURCE_CHAIN_ID --fees=100usource
```

Withdraw all rewards
```bash
sourced tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$SOURCE_CHAIN_ID --fees=100usource
```

Withdraw rewards with commision
```bash
sourced tx distribution withdraw-rewards $SOURCE_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$SOURCE_CHAIN_ID --fees=100usource
```

### Validator management
Edit validator
```bash
sourced tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$SOURCE_CHAIN_ID \
  --from=$WALLET \
  --fees=100usource -y
```

Unjail validator
```bash
sourced tx slashing unjail \
  --from=$WALLET \
  --chain-id=$SOURCE_CHAIN_ID \
  --fees=100usource -y
```

### Delete node
```bash
sudo systemctl stop sourced
sudo systemctl disable sourced
sudo rm /etc/systemd/system/source* -rf
sudo rm $(which sourced) -rf
sudo rm $HOME/.source* -rf
sudo rm $HOME/source -rf
```
