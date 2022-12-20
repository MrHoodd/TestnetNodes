# Official Links
[Website](https://www.clan.network) [Twitter](https://twitter.com/ClanNetw0rk) [Discord](https://discord.gg/gnEeUKM8TW)

# Explorer
[Explorer](https://testnet.explorer.testnet.run/Clan%20Network/staking) 

# Install Node Guide Clan Network

### Create variables
Specify the name of your moniker (validator) which will be visible in the explorer MONIKER=<YOUR_MONIKER_NAME>
```bash
MONIKER=<YOUR_MONIKER_NAME>
CHAIN="playstation-2"
WALLET_NAME=wallet
```

### Import variables into system
```bash
echo 'export MONIKER='${MONIKER} >> $HOME/.bash_profile
echo 'export CHAIN='${CHAIN} >> $HOME/.bash_profile
echo 'export WALLET_NAME='${WALLET_NAME} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Update Packages and Depencies
```bash
sudo apt update && sudo apt upgrade -y
```

### Install Depencies
```bash
sudo apt install make clang pkg-config libssl-dev build-essential git gcc chrony curl jq ncdu bsdmainutils htop net-tools lsof fail2ban wget -y
```

### Install GO
```bash
cd $HOME
wget -O go1.18.1.linux-amd64.tar.gz https://golang.org/dl/go1.18.1.linux-amd64.tar.gz
rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.18.1.linux-amd64.tar.gz && rm go1.18.1.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
go version
```

### Download binaries
```bash
git clone https://github.com/ClanNetwork/clan-network
cd clan-network
git checkout v1.0.4-alpha
make install
chmod +x /root/go/bin/cland && sudo mv /root/go/bin/cland /usr/local/bin/cland
cd $HOME
```

### Config app
```bash
cland config chain-id $CHAIN
cland config keyring-backend file
cland init $MONIKER --chain-id $CHAIN
```

### Download genesis and addrbook
```bash
curl https://raw.githubusercontent.com/ClanNetwork/testnets/main/$CHAIN/genesis.json > ~/.clan/config/genesis.json
```

### Set seeds and peers
```bash
CHAIN_REPO="https://raw.githubusercontent.com/ClanNetwork/testnets/main/$CHAIN" && \
export PEERS="$(curl -s "$CHAIN_REPO/persistent-peers.txt")"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.clan/config/config.toml
```

### Config pruning (Optional)
```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.clan/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.clan/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.clan/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.clan/config/app.toml
```

### Set minimum gas price
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uclan\"/" ~/.clan/config/app.toml
```

### Indexer (Optional)
```bash
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.clan/config/config.toml
```

### Reset chain data
```bash
cland tendermint unsafe-reset-all --home $HOME/.clan
```

### Create service
```bash
sudo tee /etc/systemd/system/cland.service > /dev/null <<EOF
[Unit]
Description=clan node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cland) start
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
sudo systemctl enable cland
sudo systemctl restart cland 
source $HOME/.bash_profile
```

### Create wallet
To create a new wallet, don't forget to save the mnemonics
```bash
cland keys add $WALLET_NAME
```
To recover existing keys use
```bash
cland keys add $WALLET_NAME --recover
```
List of wallets
```bash
cland keys list
```
### Save wallet info
Add wallet and valoper address into variables
```bash
WALLET_ADDRESS=$(cland keys show $WALLET_NAME -a)
VALOPER=$(cland keys show $WALLET_NAME --bech val -a)
echo 'export WALLET_ADDRESS='${WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export VALOPER='${VALOPER} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### To check your wallet balance:
```bash
cland query bank balances $WALLET_ADDRESS
```
### Create validator
After your node is synced, create validator
To check if your node is synced simply run
```bash
cland status 2>&1 | jq .SyncInfo
```
To create your validator run command below
```bash
cland tx staking create-validator \
  --amount 1000000000uclan \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey=$(cland tendermint show-validator) \
  --moniker $MONIKER \
  --chain-id $CHAIN \
  --fees 500uclan \
  --from $WALLET_NAME -y
```

## Usefull commands
### Service management
Check logs
```bash
journalctl -fu cland -o cat
```

Start service
```bash
systemctl start cland
```

Stop service
```bash
systemctl stop cland
```

Restart service
```bash
systemctl restart cland
```

## Node info
Synchronization info
```bash
cland status 2>&1 | jq .SyncInfo
```

Validator info
```bash
cland status 2>&1 | jq .ValidatorInfo
```

Node info
```bash
cland status 2>&1 | jq .NodeInfo
```

Show node id
```bash
cland tendermint show-node-id
```

## Wallet operations
List of wallets
```bash
cland keys list
```

Recover wallet
```bash
cland keys add $WALLET_NAME --recover
```

Delete wallet
```bash
cland keys delete $WALLET_NAME
```

Get wallet balance
```bash
cland query bank balances $WALLET_ADDRESS
```

Transfer funds
```bash
cland tx bank send $WALLET_ADDRESS <TO_WALLET_ADDRESS> 10000000uknow
```

### Voting
```bash
cland tx gov vote 1 yes --from $WALLET_NAME --chain-id=$CHAIN
```

### Staking, Delegation and Rewards
Delegate stake
```bash
cland tx staking delegate $VALOPER 1000000uclan --from $WALLET_NAME --chain-id $CHAIN --fees 500uclan
```

Redelegate stake from validator to another validator
```bash
cland tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000uclan --from=$WALLET_NAME --chain-id=$CHAIN --fees 500uclan
```

Withdraw all rewards
```bash
cland tx distribution withdraw-all-rewards --from=$WALLET_NAME --chain-id=$CHAIN --fees 500uclan
```

Withdraw rewards with commision
```bash
cland tx distribution withdraw-rewards $VALOPER --from=$WALLET_NAME --commission --chain-id=$CHAIN --fees 500uclan
```

### Validator management
Edit validator
```bash
cland tx staking edit-validator \
  --moniker=$MONIKER \
  --identity="<your_keybase_id>" \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$CHAIN \
  --from=$WALLET_NAME
  ```

Unjail validator
```bash
cland tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET_NAME \
  --chain-id=$CHAIN \
  --fees=500uclan
```
Delete node
```bash
sudo systemctl stop cland
sudo systemctl disable cland
sudo rm /etc/systemd/system/cland* -rf
sudo rm $(which cland) -rf
sudo rm $HOME/.clan* -rf
sudo rm $HOME/clan-network -rf
sed -i '/CLAN_/d' ~/.bash_profile
```
