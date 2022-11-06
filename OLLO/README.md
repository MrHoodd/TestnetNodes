# Official Links
[Website](https://linktr.ee/OllOStation) [Twitter](https://mobile.twitter.com/OllOStation) [Discord](https://discord.gg/GxBqZ9mSSm)

# Explorer
[Explorer](https://explorer.ollo.zone/ollo/staking)

# Install Node Guide ollo-testnet-1
### Setting up variables
Specify the name of your moniker (validator) which will be visible in the explorer
```bash
NODENAME=<YOUR_MONIKER_NAME>
```
### Save and import variables into system
```bash
OLLO_PORT=32
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export OLLO_CHAIN_ID=ollo-testnet-1" >> $HOME/.bash_profile
echo "export OLLO_PORT=${OLLO_PORT}" >> $HOME/.bash_profile
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
  ver="1.18.2"
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
cd $HOME && rm -rf ollo
git clone https://github.com/OLLO-Station/ollo.git
cd ollo
git checkout v0.0.1
make install
```

### Config app
```bash
ollod config chain-id $OLLO_CHAIN_ID
ollod config keyring-backend test
ollod config node tcp://localhost:${OLLO_PORT}657
```

## Init app
```bash
ollod init $NODENAME --chain-id $OLLO_CHAIN_ID
```

### Download genesis and addrbook
```bash
wget -O $HOME/.ollo/config/addrbook.json "https://raw.githubusercontent.com/BccNodes/Testnet-Guides/main/Ollo%20Testnet/addrbook.json"
```

## Set seeds and peers
```bash
SEEDS=""
PEERS="a99fc4e81770ca32d574cac2e8680dccc9b55f74@18.144.61.148:26656,70ba32724461c7ed4ec8d6ddc8b5e0b1cfb9e237@54.219.57.63:26656,7864a2e4b42e5af76a83a8b644b9172fa1e40fa5@52.8.174.235:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.ollo/config/config.toml
```

## Set custom ports
```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${OLLO_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${OLLO_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${OLLO_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${OLLO_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${OLLO_PORT}660\"%" $HOME/.ollo/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${OLLO_PORT}317\"%; s%^address = \":8080\"%address = \":${OLLO_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${OLLO_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${OLLO_PORT}091\"%" $HOME/.ollo/config/app.toml
```

### Config pruning (Optional)
```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.ollo/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.ollo/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.ollo/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.ollo/config/app.toml
```

### Set minimum gas price
```bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0utollo\"/" $HOME/.ollo/config/app.toml
```

### Enable prometheus
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.ollo/config/config.toml
```

### Reset chain data
```bash
ollod tendermint unsafe-reset-all --home $HOME/.ollo
```

### Create service
```bash
sudo tee /etc/systemd/system/ollod.service > /dev/null <<EOF
[Unit]
Description=ollo
After=network-online.target

[Service]
User=$USER
ExecStart=$(which ollod) start --home $HOME/.ollo
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
sudo systemctl enable ollod
sudo systemctl restart ollod && sudo journalctl -u ollod -f -o cat
```
### Create wallet
To create a new wallet, don't forget to save the mnemonics 
```bash
ollod keys add $WALLET
```

To recover existing keys use
```bash
ollod keys add $WALLET --recover
```

List of wallets
```bash
ollod keys list
```
### Save wallet info
Add wallet and valoper address into variables
```bash
OLLO_WALLET_ADDRESS=$(ollod keys show $WALLET -a)
OLLO_VALOPER_ADDRESS=$(ollod keys show $WALLET --bech val -a)
echo 'export OLLO_WALLET_ADDRESS='${OLLO_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export OLLO_VALOPER_ADDRESS='${OLLO_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### To check your wallet balance:
```bash
ollod query bank balances $OLLO_WALLET_ADDRESS
```
### Create validator
After your node is synced, create validator
```bash
ollod tx staking create-validator \
  --amount 50000000utollo \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(ollod tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $OLLO_CHAIN_ID
```

## Usefull commands
### Service management
Check logs
```bash
journalctl -fu ollod -o cat
```

Start service
```bash
sudo systemctl start ollod
```

Stop service
```bash
sudo systemctl stop ollod
```

Restart service
```bash
sudo systemctl restart ollod
```

### Node info
Synchronization info
```bash
ollod status 2>&1 | jq .SyncInfo
```

Validator info
```bash
ollod status 2>&1 | jq .ValidatorInfo
```

Node info
```bash
ollod status 2>&1 | jq .NodeInfo
```

Show node id
```bash
ollod tendermint show-node-id
```

### Wallet operations
List of wallets
```bash
ollod keys list
```

Recover wallet
```bash
ollod keys add $WALLET --recover
```

Delete wallet
```bash
ollod keys delete $WALLET
```

Get wallet balance
```bash
ollod query bank balances $OLLO_WALLET_ADDRESS
```

Transfer funds
```bash
ollod tx bank send $OLLO_WALLET_ADDRESS <TO_OLLO_WALLET_ADDRESS> 10000000utollo
```

### Voting
```bash
ollod tx gov vote 1 yes --from $WALLET --chain-id=$OLLO_CHAIN_ID
```

### Staking, Delegation and Rewards
Delegate stake
```bash
ollod tx staking delegate $OLLO_VALOPER_ADDRESS 10000000utollo --from=$WALLET --chain-id=$OLLO_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```bash
ollod tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000utollo --from=$WALLET --chain-id=$OLLO_CHAIN_ID --gas=auto
```

Withdraw all rewards
```bash
ollod tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$OLLO_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```bash
ollod tx distribution withdraw-rewards $OLLO_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$OLLO_CHAIN_ID
```

### Validator management
Edit validator
```bash
ollod tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$OLLO_CHAIN_ID \
  --from=$WALLET
```

Unjail validator
```bash
ollod tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$OLLO_CHAIN_ID \
  --gas=auto
```

### Delete node
```bash
sudo systemctl stop ollod
sudo systemctl disable ollod
sudo rm /etc/systemd/system/ollo* -rf
sudo rm $(which ollod) -rf
sudo rm $HOME/.ollo* -rf
sudo rm $HOME/ollo -rf
sed -i '/OLLO_/d' ~/.bash_profile
```

