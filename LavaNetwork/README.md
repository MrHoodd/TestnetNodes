# Official Links
[Website](https://lavanet.xyz/) [Twitter](https://twitter.com/lavanetxyz) [Discord](https://discord.gg/5VcqgwMmkA)

## [Explorer](http://explorer.nodera.org/lava/staking) [Explorer](https://explorer.stavr.tech/lava-testnet/staking)

# Install Node Lava Network 
### Setting up variables

Specify the name of your moniker (validator) which will be visible in the explorer
```bash
NODENAME=<YOUR_MONIKER_NAME>
```
### Save and import variables into system
```bash
LAVA_PORT=37
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export LAVA_CHAIN_ID=lava-testnet-1" >> $HOME/.bash_profile
echo "export LAVA_PORT=${LAVA_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Update Packages and Depencies
```bash
sudo apt update && sudo apt upgrade -y
```

### Install Depencies
```bash
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
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
cd $HOME 
curl -L https://lava-binary-upgrades.s3.amazonaws.com/testnet/v0.4.0/lavad > lavad
chmod +x lavad
sudo mv lavad /usr/local/bin/lavad
```

### Config app
```bash
lavad config chain-id $LAVA_CHAIN_ID
lavad config keyring-backend test
lavad config node tcp://localhost:${LAVA_PORT}657
```

## Init app
```bash
humansd init $NODENAME --chain-id $LAVA_CHAIN_ID
```

### Download genesis and addrbook
```bash
curl -s https://raw.githubusercontent.com/K433QLtr6RA9ExEq/GHFkqmTzpdNLDd6T/main/testnet-1/genesis_json/genesis.json > $HOME/.lava/config/genesis.json
curl -s https://snapshots1-testnet.nodejumper.io/lava-testnet/addrbook.json > $HOME/.lava/config/addrbook.json
```

## Set seeds and peers
```bash
SEEDS="3a445bfdbe2d0c8ee82461633aa3af31bc2b4dc0@prod-pnet-seed-node.lavanet.xyz:26656,e593c7a9ca61f5616119d6beb5bd8ef5dd28d62d@prod-pnet-seed-node2.lavanet.xyz:26656"
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.lava/config/config.toml
```

### Set minimum gas price
```bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025ulava\"/" $HOME/.lava/config/app.toml
```

## Set custom ports
```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${LAVA_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${LAVA_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${LAVA_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${LAVA_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${LAVAPORT}660\"%" $HOME/.lava/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${LAVA_PORT}317\"%; s%^address = \":8080\"%address = \":${LAVAPORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${LAVA_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${LAVA_PORT}091\"%" $HOME/.lava/config/app.toml
```

### Config pruning (Optional)
```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.lava/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.lava/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.lava/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.lava/config/app.toml
```

### Indexer (Optional)
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.humans/config/config.toml
```

### Enable prometheus (Optional)
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.lava/config/config.toml
```

### Reset chain data
```bash
lavad tendermint unsafe-reset-all --home $HOME/.lava
```

### Create service
```bash
sudo tee /etc/systemd/system/lavad.service > /dev/null <<EOF
[Unit]
Description=lava
After=network-online.target

[Service]
User=$USER
ExecStart=$(which lavad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### Register and start service
```bash
sudo systemctl daemon-reload && \
sudo systemctl enable lavad && \
sudo systemctl restart lavad && sudo journalctl -u lavad -f -o cat
```

### Snapshot by NodeJumper
```
LAVA_SNAP=$(curl -s https://snapshots1-testnet.nodejumper.io/lava-testnet/ | egrep -o ">lava-testnet-1.*\.tar.lz4" | tr -d ">")
curl https://snapshots1-testnet.nodejumper.io/lava-testnet/${LAVA_SNAP} | lz4 -dc - | tar -xf - -C $HOME/.lava
sudo systemctl restart lavad && sudo journalctl -u lavad -f -o cat
```

### Create wallet
To create a new wallet, don't forget to save the mnemonics 
```bash
lavad keys add $WALLET
```

To recover existing keys use
```bash
lavad keys add $WALLET --recover
```

List of wallets
```bash
lavad keys list
```

### Save wallet info
Add wallet and valoper address into variables
```bash
LAVA_WALLET_ADDRESS=$(lavad keys show $WALLET -a)
LAVA_VALOPER_ADDRESS=$(lavad keys show $WALLET --bech val -a)
echo 'export LAVA_WALLET_ADDRESS='${LAVA_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export LAVA_VALOPER_ADDRESS='${LAVA_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### To check your wallet balance:
```bash
lavad query bank balances $LAVA_WALLET_ADDRESS
```

### Create validator
After your node is synced, create validator
```bash
lavad tx staking create-validator \
  --amount 10000000ulava \
  --from $WALLET \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(lavad tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $LAVA_CHAIN_ID \
  --fees=10000ulava \
  --identity="" \
  --details="" \
  --website="" -y
```

## Usefull commands
### Service management
Check logs
```bash
journalctl -fu lavad -o cat
```

Start service
```bash
sudo systemctl start lavad
```

Stop service
```bash
sudo systemctl stop lavad
```

Restart service
```bash
sudo systemctl restart lavad
```

### Node info
Synchronization info
```bash
lavad status 2>&1 | jq .SyncInfo
```

Validator info
```bash
lavad status 2>&1 | jq .ValidatorInfo
```

Node info
```bash
lavad status 2>&1 | jq .NodeInfo
```

Show node id
```bash
lavad tendermint show-node-id
```

### Wallet operations
List of wallets
```bash
lavad keys list
```

Recover wallet
```bash
lavad keys add $WALLET --recover
```

Delete wallet
```bash
lavad keys delete $WALLET
```

Get wallet balance
```bash
lavad query bank balances $LAVA_WALLET_ADDRESS
```

Transfer funds
```bash
lavad tx bank send $LAVA_WALLET_ADDRESS <TO_OLLO_WALLET_ADDRESS> 10000000ulava 
```

### Voting
```bash
lavad tx gov vote 1 yes --from $WALLET --chain-id=$LAVA_CHAIN_ID --fees=10000ulava -y
```

### Staking, Delegation and Rewards
Delegate stake
```bash
lavad tx staking delegate $LAVA_VALOPER_ADDRESS 10000000ulava --from=$WALLET --chain-id=$LAVA_CHAIN_ID --fees=10000ulava -y
```

Redelegate stake from validator to another validator
```bash
lavad tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000ulava --from=$WALLET --chain-id=$LAVA_CHAIN_ID --fees=10000ulava 
```

Withdraw all rewards
```bash
lavad tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$LAVA_CHAIN_ID --fees=10000ulava -y
```

Withdraw rewards with commision
```bash
lavad tx distribution withdraw-rewards $LAVA_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$LAVA_CHAIN_ID --fees=10000ulava -y
```

### Validator management
Edit validator
```bash
lavad tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$LAVA_CHAIN_ID \
  --from=$WALLET \
  --fees=10000ulava -y
```

Unjail validator
```bash
lavad tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$LAVA_CHAIN_ID \
  --fees=10000ulava -y
```

### Delete node
```bash
sudo systemctl stop lavad && \
sudo systemctl disable lavad && \
rm /etc/systemd/system/lavad.service && \
sudo rm -rf /usr/local/bin/lavad && \
sudo systemctl daemon-reload && \
cd $HOME && \
sudo rm -rf $HOME/.lava
sudo rm -rf $HOME/lavad
rm -rf $(which lavad)
sed -i '/LAVA_/d' ~/.bash_profile
```
