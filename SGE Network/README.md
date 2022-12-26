# Official Links
[Website](https://sixsigmasports.io/) [Twitter](https://twitter.com/SixSigmaSports) [Discord](https://discord.gg/gC4AHWyu3k)

## [Explorer](https://blockexplorer.testnet.sgenetwork.io/validators) [Explorer](https://explorer.stavr.tech/sge-testnet/staking)
## [App](https://testnet.sixsigmasports.io/matches)


# Install Node Guide SGE Network

### Setting up variables
Specify the name of your moniker (validator) which will be visible in the explorer
```bash
NODENAME=<YOUR_MONIKER_NAME>
```

### Save and import variables into system
```bash
SGE_PORT=27
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export SGE_CHAIN_ID=sge-testnet-1" >> $HOME/.bash_profile
echo "export SGE_PORT=${SGE_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Update Packages and Depencies
```bash
sudo apt update && sudo apt upgrade -y
```

### Install Depencies
```bash
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git chrony liblz4-tool fail2ban -y
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
git clone https://github.com/sge-network/sge
git clone https://github.com/sge-network/networks
cd sge
git fetch --tags
git checkout v0.0.3
cd sge
go mod tidy
make install
```

### Config app
```bash
sged config chain-id $SGE_CHAIN_ID
sged config keyring-backend file
sged init $NODENAME --chain-id $SGE_CHAIN_ID
```

### Download genesis and addrbook
```bash
wget -O $HOME/.sge/config/genesis.json "https://raw.githubusercontent.com/sge-network/networks/master/sge-testnet-1/genesis.json"
wget -O $HOME/.sge/config/addrbook.json "https://raw.githubusercontent.com/obajay/nodes-Guides/main/SGE/addrbook.json"
```

### Set seeds and peers minimum gas price filter peers maxpeers
```bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0usge\"/" $HOME/.sge/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.sge/config/config.toml
external_address=$(wget -qO- eth0.me) 
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.sge/config/config.toml
peers="27f0b281ea7f4c3db01fdb9f4cf7cc910ad240a6@209.34.206.44:26656,5f3196f370fa865bfd3e4a0653dc7853f613aba6@[2a01:4f9:1a:a718::10]:26656,afa90de6a195a4a2993b2501f12a1cd306f01d02@136.243.103.32:60856,dc75f5d2f9458767f39f62bd7eab3f499fdf2761@104.248.236.171:26656,1168931936c638e92ea6d93e2271b3fe5faee6d1@51.91.145.100:26656,8a7d722dba88326ee69fcc23b5b2ac93e36d7ff2@65.108.225.158:17756,445506c736895336e36dd4f8228a60c257b30e61@20.12.75.0:26656,971643c5b9f9d279cfb7ac1b14accd109231236b@65.108.15.170:26656,788bb7ee73c023f70c41360e9014544b12fe23f9@3.15.209.96:26656,26f0965f8cd53f2b3adc26f8ca5e893929b66c15@52.44.14.245:26656,4a3f59e30cde63d00aed8c3d15bef46b34ec2c7f@50.19.180.153:26656,31d742df5a427e241d1a6b1b22813c9cb4888c07@65.21.181.169:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.sge/config/config.toml
seeds=""
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.sge/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.sge/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.sge/config/config.toml

```

### Config pruning (Optional)
```bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.sge/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.sge/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.sge/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.sge/config/app.toml
```

### Indexer (Optional)
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.sge/config/config.toml
```

### Set custom ports
```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:2${SGE_PORT}8\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:2${SGE_PORT}7\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${SGE_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:2${SGE_PORT}6\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":2${SGE_PORT}0\"%" $HOME/.sge/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${SGE_PORT}7\"%; s%^address = \":8080\"%address = \":${SGE_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${SGE_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${SGE_PORT}91\"%" $HOME/.sge/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:2${SGE_PORT}7\"%" $HOME/.sge/config/client.toml
```

### Reset chain data
```bash
sged tendermint unsafe-reset-all --home $HOME/.sge
```

### Create service
```bash
sudo tee /etc/systemd/system/sged.service > /dev/null <<EOF
[Unit]
Description=sge
After=network-online.target

[Service]
User=$USER
ExecStart=$(which sged) start
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
sudo systemctl enable sged
sudo systemctl restart sged && sudo journalctl -u sged -f -o cat
```

### Create wallet
To create a new wallet, don't forget to save the mnemonics
```bash
sged keys add $WALLET
```
To recover existing keys use
```bash
sged keys add $WALLET --recover
```
List of wallets
```bash
sged keys list
```
### Save wallet info
Add wallet and valoper address into variables
```bash
SGE_WALLET_ADDRESS=$(sged keys show $WALLET -a)
SGE_VALOPER_ADDRESS=$(sged keys show $WALLET --bech val -a)
echo 'export SGE_WALLET_ADDRESS='${SGE_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export SGE_VALOPER_ADDRESS='${SGE_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### To check your wallet balance:
```bash
sged query bank balances $SGE_WALLET_ADDRESS
```
### Create validator
After your node is synced, create validator
To check if your node is synced simply run
```bash
sged status 2>&1 | jq .SyncInfo
```
To create your validator run command below
```bash
sged tx staking create-validator \
  --amount 1000000000usge \
  --from $WALLET \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(sged tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $SGE_CHAIN_ID \
  --identity="" \
  --details="" \
  --website="" -y
```

## Usefull commands
### Service management
Check logs
```bash
journalctl -fu sged -o cat
```

Start service
```bash
systemctl start sged
```

Stop service
```bash
systemctl stop sged
```

Restart service
```bash
systemctl restart sged
```

## Node info
Synchronization info
```bash
sged status 2>&1 | jq .SyncInfo
```

Validator info
```bash
sged status 2>&1 | jq .ValidatorInfo
```

Node info
```bash
sged status 2>&1 | jq .NodeInfo
```

Show node id
```bash
sged tendermint show-node-id
```

## Wallet operations
List of wallets
```bash
sged keys list
```

Recover wallet
```bash
sged keys add $WALLET --recover
```

Delete wallet
```bash
sged keys delete $WALLET
```

Get wallet balance
```bash
sged query bank balances $SGE_WALLET_ADDRESS
```

Transfer funds
```bash
sged tx bank send $SGE_WALLET_ADDRESS <TO_WALLET_ADDRESS> 10000000usge
```

### Voting
```bash
sged tx gov vote 1 yes --from $WALLET --chain-id=$SGE_CHAIN_ID 
```

### Staking, Delegation and Rewards
Delegate stake
```bash
sged tx staking delegate SGE_VALOPER_ADDRESS 1000000usge --from $WALLET --chain-id $SGE_CHAIN_ID 
```

Redelegate stake from validator to another validator
```bash
sged tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000usge --from=$WALLET --chain-id=$SGE_CHAIN_ID 
```

Withdraw all rewards
```bash
sged tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$SGE_CHAIN_ID 
```

Withdraw rewards with commision
```bash
sged tx distribution withdraw-rewards SGE_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$SGE_CHAIN_ID 
```

### Validator management
Edit validator
```bash
sged tx staking edit-validator \
  --moniker=$NODENAME \
  --identity="<your_keybase_id>" \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$SGE_CHAIN_ID \
  --from=$WALLET
  ```

Unjail validator
```bash
sged tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$SGE_CHAIN_ID 
```
Delete node
```bash
sudo systemctl stop sged && \
sudo systemctl disable sged && \
rm /etc/systemd/system/sged.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf sge && \
rm -rf .sge && \
rm -rf $(which sged)
sed -i '/SGE_/d' ~/.bash_profile
```
