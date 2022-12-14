# Official Links
[Website](https://crowdcontrol.network/#/) [Twitter](https://twitter.com/CrowdControlNet?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E1577353712963665922%7Ctwgr%5Eeffb4472653f0071a6ba1c0a52b2f82372d78fc6%7Ctwcon%5Es1_&ref_url=https%3A%2F%2Flinktr.ee%2Fcrowdcontrolnet) [Discord](https://discord.gg/yPA3aKe)

# Explorer
[Explorer](https://explorer.kjnodes.com/cardchain/staking)

# Install Node Guide Crowd Control
### Setting up variables
Specify the name of your moniker (validator) which will be visible in the explorer
```bash
NODENAME=<YOUR_MONIKER_NAME>
```
### Save and import variables into system
```bash
CARDCHAIN_PORT=18
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export CARDCHAIN_CHAIN_ID=Testnet3" >> $HOME/.bash_profile
echo "export CARDCHAIN_PORT=${CARDCHAIN_PORT}" >> $HOME/.bash_profile
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
wget https://github.com/DecentralCardGame/Cardchain/releases/download/v0.81/Cardchain_latest_linux_amd64.tar.gz
tar xzf Cardchain_latest_linux_amd64.tar.gz
chmod 775 Cardchaind
sudo mv Cardchaind /usr/local/bin/
sudo rm Cardchain_latest_linux_amd64.tar.gz
```

### Config app
```bash
Cardchaind config chain-id $CARDCHAIN_CHAIN_ID
Cardchaind config keyring-backend test
Cardchaind config node tcp://localhost:${CARDCHAIN_PORT}657
```

## Init app
```bash
Cardchaind init $NODENAME --chain-id $CARDCHAIN_CHAIN_ID
```

### Download genesis and addrbook
```bash
wget -O $HOME/.Cardchain/config/genesis.json https://raw.githubusercontent.com/DecentralCardGame/Testnet/main/genesis.json
```

## Set seeds and peers
```bash
SEEDS=""
PEERS="56d11635447fa77163f31119945e731c55e256a4@45.136.28.158:26658"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.Cardchain/config/config.toml
```

## Set custom ports
```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CARDCHAIN_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CARDCHAIN_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CARDCHAIN_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CARDCHAIN_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CARDCHAIN_PORT}660\"%" $HOME/.Cardchain/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CARDCHAIN_PORT}317\"%; s%^address = \":8080\"%address = \":${CARDCHAIN_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CARDCHAIN_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CARDCHAIN_PORT}091\"%" $HOME/.Cardchain/config/app.toml
```

### Config pruning (Optional)
```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.Cardchain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.Cardchain/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.Cardchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.Cardchain/config/app.toml
```

### Set minimum gas price
```bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ubpf\"/" $HOME/.Cardchain/config/app.toml
```

### Enable prometheus
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.Cardchain/config/config.toml
```

### Reset chain data
```bash
Cardchaind unsafe-reset-all --home $HOME/.Cardchain
```

### Create service
```bash
sudo tee /etc/systemd/system/Cardchaind.service > /dev/null <<EOF
[Unit]
Description=Cardchain
After=network-online.target

[Service]
User=$USER
ExecStart=$(which Cardchaind) start --home $HOME/.Cardchain
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
sudo systemctl enable Cardchaind
sudo systemctl restart Cardchaind && sudo journalctl -u Cardchaind -f -o cat
```
### Create wallet
To create a new wallet, don't forget to save the mnemonics 
```bash
Cardchaind keys add $WALLET
```

To recover existing keys use
```bash
Cardchaind keys add $WALLET --recover
```

List of wallets
```bash
Cardchaind keys list
```
### Save wallet info
Add wallet and valoper address into variables
```bash
CARDCHAIN_WALLET_ADDRESS=$(Cardchaind keys show $WALLET -a)
CARDCHAIN_VALOPER_ADDRESS=$(Cardchaind keys show $WALLET --bech val -a)
echo 'export CARDCHAIN_WALLET_ADDRESS='${CARDCHAIN_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export CARDCHAIN_VALOPER_ADDRESS='${CARDCHAIN_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### To check your wallet balance:
```bash
Cardchaind query bank balances $CARDCHAIN_WALLET_ADDRESS
```
### Create validator
After your node is synced, create validator
```bash
Cardchaind tx staking create-validator \
  --amount 1000000ubpf \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(Cardchaind tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $CARDCHAIN_CHAIN_ID
```

## Usefull commands
### Service management
Check logs
```bash
journalctl -fu Cardchaind -o cat
```

Start service
```bash
sudo systemctl start Cardchaind
```

Stop service
```bash
sudo systemctl stop Cardchaind
```

Restart service
```bash
sudo systemctl restart Cardchaind
```

### Node info
Synchronization info
```bash
Cardchaind status 2>&1 | jq .SyncInfo
```

Validator info
```bash
Cardchaind status 2>&1 | jq .ValidatorInfo
```

Node info
```bash
Cardchaind status 2>&1 | jq .NodeInfo
```

Show node id
```bash
Cardchaind tendermint show-node-id
```

### Wallet operations
List of wallets
```bash
Cardchaind keys list
```

Recover wallet
```bash
Cardchaind keys add $WALLET --recover
```

Delete wallet
```bash
Cardchaind keys delete $WALLET
```

Get wallet balance
```bash
Cardchaind query bank balances $CARDCHAIN_WALLET_ADDRESS
```
### Voting
```bash
Cardchaind tx gov vote 1 yes --from $WALLET --chain-id=$CARDCHAIN_CHAIN_ID
```

### Staking, Delegation and Rewards
Delegate stake
```bash
Cardchaind tx staking delegate $CARDCHAIN_VALOPER_ADDRESS 10000000ubpf --from=$WALLET --chain-id=$CARDCHAIN_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```bash
Cardchaind tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000ubpf --from=$WALLET --chain-id=$CARDCHAIN_CHAIN_ID --gas=auto
```

Withdraw all rewards
```bash
Cardchaind tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$CARDCHAIN_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```bash
Cardchaind tx distribution withdraw-rewards $CARDCHAIN_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$CARDCHAIN_CHAIN_ID
```
Transfer funds
```bash
Cardchaind tx bank send $CARDCHAIN_WALLET_ADDRESS <TO_CARDCHAIN_WALLET_ADDRESS> 10000000ubpf
```

### Validator management
Edit validator
```bash
Cardchaind tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$CARDCHAIN_CHAIN_ID \
  --from=$WALLET
```

Unjail validator
```bash
Cardchaind tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$CARDCHAIN_CHAIN_ID \
  --gas=auto
```

### Delete node
```bash
sudo systemctl stop Cardchaind
sudo systemctl disable Cardchaind
sudo rm /etc/systemd/system/Cardchain* -rf
sudo rm $(which Cardchaind) -rf
sudo rm $HOME/.Cardchain* -rf
sudo rm $HOME/Cardchain -rf
sed -i '/CARDCHAIN_/d' ~/.bash_profile
```
