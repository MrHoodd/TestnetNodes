# Official Links
[Website](https://www.clan.network) [Twitter](https://twitter.com/ClanNetw0rk) [Discord](https://discord.gg/gnEeUKM8TW)

# Explorer
[Explorer](https://testnet.explorer.testnet.run/Clan%20Network/staking) 

# Install Node Guide Clan Network

### Setting up variables
Specify the name of your moniker (validator) which will be visible in the explorer
```bash
NODENAME=<YOUR_MONIKER_NAME>
```

### Save and import variables into system
```bash
CLAN_PORT=19
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export CLAN_CHAIN_ID=playstation-2" >> $HOME/.bash_profile
echo "export CLAN_PORT=${CLAN_PORT}" >> $HOME/.bash_profile
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
git clone https://github.com/ClanNetwork/clan-network
cd clan-network
git checkout v1.0.4-alpha
make install
chmod +x /root/go/bin/cland && sudo mv /root/go/bin/cland /usr/local/bin/cland
cd $HOME
```

### Config app
```bash
cland config chain-id $CLAN_CHAIN_ID
cland config keyring-backend file
cland init $NODENAME --chain-id $CLAN_CHAIN_ID
```

### Download genesis and addrbook
```bash
wget -O $HOME/.clan/config/genesis.json "https://raw.githubusercontent.com/ClanNetwork/testnets/main/playstation-2/genesis.json"
curl https://github.com/MrHoodd/TestnetNodes/blob/main/ClanNetwork/addrbook.json > ~/.clan/config/addrbook.json
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
sudo systemctl daemon-reload \
sudo systemctl enable cland \
sudo systemctl restart cland && journalctl -fu cland -o cat
source $HOME/.bash_profile
```

### Create wallet
To create a new wallet, don't forget to save the mnemonics
```bash
cland keys add $WALLET
```
To recover existing keys use
```bash
cland keys add $WALLET --recover
```
List of wallets
```bash
cland keys list
```
### Save wallet info
Add wallet and valoper address into variables
```bash
CLAN_WALLET_ADDRESS=$(cland keys show $WALLET -a)
CLAN_VALOPER_ADDRESS=$(cland keys show $WALLET --bech val -a)
echo 'export CLAN_WALLET_ADDRESS='${CLAN_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export CLAN_VALOPER_ADDRESS='${CLAN_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### To check your wallet balance:
```bash
cland query bank balances $CLAN_WALLET_ADDRESS
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
  --moniker $NODENAME \
  --chain-id CLAN_CHAIN_ID \
  --fees 500uclan \
  --from $WALLET -y
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
cland keys add $WALLET --recover
```

Delete wallet
```bash
cland keys delete $WALLET
```

Get wallet balance
```bash
cland query bank balances $CLAN_WALLET_ADDRESS
```

Transfer funds
```bash
cland tx bank send $CLAN_WALLET_ADDRESS <TO_WALLET_ADDRESS> 10000000uknow
```

### Voting
```bash
cland tx gov vote 1 yes --from $WALLET --chain-id=$CLAN_CHAIN_ID --fees=500uclan -y
```

### Staking, Delegation and Rewards
Delegate stake
```bash
cland tx staking delegate CLAN_VALOPER_ADDRESS 1000000uclan --from $WALLET --chain-id $CHAIN --fees 500uclan
```

Redelegate stake from validator to another validator
```bash
cland tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000uclan --from=$WALLET --chain-id=$CLAN_CHAIN_ID --fees 500uclan -y
```

Withdraw all rewards
```bash
cland tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$CLAN_CHAIN_ID --fees 500uclan -y
```

Withdraw rewards with commision
```bash
cland tx distribution withdraw-rewards CLAN_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$CLAN_CHAIN_ID --fees 500uclan -y
```

### Validator management
Edit validator
```bash
cland tx staking edit-validator \
  --moniker=$NODENAME \
  --identity="<your_keybase_id>" \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$CLAN_CHAIN_ID \
  --from=$WALLET
  ```

Unjail validator
```bash
cland tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$CLAN_CHAIN_ID \
  --fees=500uclan -y
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
