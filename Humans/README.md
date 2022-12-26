# Official Links
[Website](https://humans.ai/) [Twitter](https://twitter.com/humansdotai) [Discord](https://discord.gg/humansdotai)

## [Explorer](https://explorer.humans.zone/humans-testnet/staking) [Explorer](https://explorer.stavr.tech/humans-testnet/staking)

# Install Node Guide Humans
### Setting up variables

Specify the name of your moniker (validator) which will be visible in the explorer
```bash
NODENAME=<YOUR_MONIKER_NAME>
```
### Save and import variables into system
```bash
HUMANS_PORT=49
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export HUMANS_CHAIN_ID=testnet-1" >> $HOME/.bash_profile
echo "export HUMANS_PORT=${HUMANS_PORT}" >> $HOME/.bash_profile
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
git clone https://github.com/humansdotai/humans
cd humans
git checkout v1
go build -o humansd cmd/humansd/main.go
mv humansd /root/go/bin/humansd
```

### Config app
```bash
humansd config chain-id $HUMANS_CHAIN_ID
humansd config keyring-backend test
humansd config node tcp://localhost:${HUMANS_PORT}657
```

## Init app
```bash
humansd init $NODENAME --chain-id $HUMANS_CHAIN_ID
```

### Download genesis and addrbook
```bash
wget https://snapshots.polkachu.com/testnet-genesis/humans/genesis.json -O $HOME/.humans/config/genesis.json
wget -O $HOME/.humans/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/Humans/addrbook.json"
```

## Set seeds and peers
```bash
SEEDS=""
PEERS="1df6735ac39c8f07ae5db31923a0d38ec6d1372b@45.136.40.6:26656,9726b7ba17ee87006055a9b7a45293bfd7b7f0fc@45.136.40.16:26656,6e84cde074d4af8a9df59d125db3bf8d6722a787@45.136.40.18:26656,eda3e2255f3c88f97673d61d6f37b243de34e9d9@45.136.40.13:26656,4de8c8acccecc8e0bed4a218c2ef235ab68b5cf2@45.136.40.12:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.humans/config/config.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.humans/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.humans/config/config.toml
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.humans/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 100/g' $HOME/.humans/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 100/g' $HOME/.humans/config/config.toml

```

### Set minimum gas price
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uheart\"/;" ~/.humans/config/app.toml
```

## Set custom ports
```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${HUMANS_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${HUMANS_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${HUMANS_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${HUMANS_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${HUMANS_PORT}660\"%" $HOME/.humans/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${HUMANS_PORT}317\"%; s%^address = \":8080\"%address = \":${HUMANS_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${HUMANS_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${HUMANS_PORT}091\"%" $HOME/.humans/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:2${HUMANS_PORT}7\"%" $HOME/.humans/config/client.toml
```

### Config pruning (Optional)
```bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.humans/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.humans/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.humans/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.humans/config/app.toml
```

### Indexer (Optional)
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.humans/config/config.toml
```

### Enable prometheus
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.humans/config/config.toml
```

### Reset chain data
```bash
humansd tendermint unsafe-reset-all --home $HOME/.humans
```

### Create service
```bash
sudo tee /etc/systemd/system/humansd.service > /dev/null <<EOF
[Unit]
Description=humans
After=network-online.target

[Service]
User=$USER
ExecStart=$(which humansd) start
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
sudo systemctl enable humansd \
sudo systemctl restart humansd && sudo journalctl -u humansd -f -o cat
```

### [Snapshot](https://polkachu.com/testnets/humans/snapshots)

### Create wallet
To create a new wallet, don't forget to save the mnemonics 
```bash
humansd keys add $WALLET
```

To recover existing keys use
```bash
humansd keys add $WALLET --recover
```

List of wallets
```bash
humansd keys list
```

### Save wallet info
Add wallet and valoper address into variables
```bash
HUMANS_WALLET_ADDRESS=$(humansd keys show $WALLET -a)
HUMANS_VALOPER_ADDRESS=$(humansd keys show $WALLET --bech val -a)
echo 'export HUMANS_WALLET_ADDRESS='${HUMANS_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export HUMANS_VALOPER_ADDRESS='${HUMANS_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### To check your wallet balance:
```bash
humansd query bank balances $HUMANS_WALLET_ADDRESS
```

### Create validator
After your node is synced, create validator
```bash
humansd tx staking create-validator \
  --amount 10000000uheart \
  --from $WALLET \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(humansd tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $HUMANS_CHAIN_ID \
  --identity="" \
  --details="" \
  --website="" -y
```

## Usefull commands
### Service management
Check logs
```bash
journalctl -fu humansd -o cat
```

Start service
```bash
sudo systemctl start humansd
```

Stop service
```bash
sudo systemctl stop humansd
```

Restart service
```bash
sudo systemctl restart humansd
```

### Node info
Synchronization info
```bash
humansd status 2>&1 | jq .SyncInfo
```

Validator info
```bash
humansd status 2>&1 | jq .ValidatorInfo
```

Node info
```bash
humansd status 2>&1 | jq .NodeInfo
```

Show node id
```bash
humansd tendermint show-node-id
```

### Wallet operations
List of wallets
```bash
humansd keys list
```

Recover wallet
```bash
humansd keys add $WALLET --recover
```

Delete wallet
```bash
humansd keys delete $WALLET
```

Get wallet balance
```bash
humansd query bank balances $HUMANS_WALLET_ADDRESS
```

Transfer funds
```bash
humansd tx bank send $HUMANS_WALLET_ADDRESS <TO_OLLO_WALLET_ADDRESS> 10000000uheart
```

### Voting
```bash
humansd tx gov vote 1 yes --from $WALLET --chain-id=$HUMANS_CHAIN_ID 
```

### Staking, Delegation and Rewards
Delegate stake
```bash
humansd tx staking delegate $HUMANS_VALOPER_ADDRESS 10000000uheart --from=$WALLET --chain-id=$HUMANS_CHAIN_ID 
```

Redelegate stake from validator to another validator
```bash
humansd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000uheart --from=$WALLET --chain-id=$HUMANS_CHAIN_ID 
```

Withdraw all rewards
```bash
humansd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$HUMANS_CHAIN_ID 
```

Withdraw rewards with commision
```bash
humansd tx distribution withdraw-rewards $HUMANS_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$HUMANS_CHAIN_ID 
```

### Validator management
Edit validator
```bash
humansd tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$HUMANS_CHAIN_ID \
  --from=$WALLET 
```

Unjail validator
```bash
humansd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$HUMANS_CHAIN_ID 
```

### Delete node
```bash
sudo systemctl stop humansd && \
sudo systemctl disable humansd && \
rm /etc/systemd/system/humansd.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf humans && \
rm -rf .humans && \
rm -rf $(which humansd)
sed -i '/HUMANS_/d' ~/.bash_profile
```
