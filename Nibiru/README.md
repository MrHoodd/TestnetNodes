# Official Links
[Website](https://nibiru.fi/) [Twitter](https://twitter.com/NibiruChain) [Discord](https://discord.gg/bFamKuv9)

# Explorer
[Explorer](https://explorer.kjnodes.com/nibiru/staking) [Explorer](https://testnet-1.nibiru.fi/validators)

# Install Node Guide nibiru-testnet-2
### Setting up variables
Specify the name of your moniker (validator) which will be visible in the explorer
```bash
NODENAME=<YOUR_MONIKER_NAME>
```
### Save and import variables into system
```bash
NIBIRU_PORT=39
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export NIBIRU_CHAIN_ID=nibiru-testnet-2" >> $HOME/.bash_profile
echo "export NIBIRU_PORT=${NIBIRU_PORT}" >> $HOME/.bash_profile
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
cd $HOME && rm -rf nibiru
git clone https://github.com/NibiruChain/nibiru.git
cd nibiru
git checkout v0.16.2
make install
```

### Config app
```bash
nibid config chain-id $NIBIRU_CHAIN_ID
nibid config keyring-backend test
nibid config node tcp://localhost:${NIBIRU_PORT}657
```

## Init app
```bash
nibid init $NODENAME --chain-id $NIBIRU_CHAIN_ID
```

### Download genesis and addrbook
```bash
curl -s https://networks.testnet.nibiru.fi/nibiru-testnet-2/genesis > $HOME/.nibid/config/genesis.json
```

## Set seeds and peers
```bash
sed -i 's|seeds =.*|seeds = "'$(curl -s https://networks.testnet.nibiru.fi/nibiru-testnet-2/seeds)'"|g' $HOME/.nibid/config/config.toml
```

## Set custom ports
```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${NIBIRU_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${NIBIRU_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${NIBIRU_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${NIBIRU_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${NIBIRU_PORT}660\"%" $HOME/.nibid/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${NIBIRU_PORT}317\"%; s%^address = \":8080\"%address = \":${NIBIRU_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${NIBIRU_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${NIBIRU_PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${NIBIRU_PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${NIBIRU_PORT}546\"%" $HOME/.nibid/config/app.toml
```

### Config pruning (Optional)
```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.nibid/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.nibid/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.nibid/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.nibid/config/app.toml
```

### Set minimum gas price
```bash
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.025unibi"/g' $HOME/.nibid/config/app.toml
```

### Enable prometheus
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.nibid/config/config.toml
```

### Reset chain data
```bash
nibid tendermint unsafe-reset-all --home $HOME/.nibid
```

### Create service
```bash
sudo tee /etc/systemd/system/nibid.service > /dev/null <<EOF
[Unit]
Description=nibi
After=network-online.target

[Service]
User=$USER
ExecStart=$(which nibid) start --home $HOME/.nibid
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
sudo systemctl enable nibid
sudo systemctl restart nibid && sudo journalctl -u nibid -f -o cat
```
### Snapshot
```bash
sudo systemctl stop nibid
cp $HOME/.nibid/data/priv_validator_state.json $HOME/.nibid/priv_validator_state.json.backup
rm -rf $HOME/.nibid/data
curl -L https://snapshots.kjnodes.com/nibiru-testnet/snapshot_latest.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.nibid
mv $HOME/.nibid/priv_validator_state.json.backup $HOME/.nibid/data/priv_validator_state.json
sudo systemctl start nibid && journalctl -u nibid -f --no-hostname -o cat
```
### Create wallet
To create a new wallet, don't forget to save the mnemonics
```bash
nibid keys add $WALLET
```

To recover existing keys use
```bash
nibid keys add $WALLET --recover
```

List of wallets
```bash
nibid keys list
```

### Save wallet info
Add wallet and valoper address into variables
```bash
NIBIRU_WALLET_ADDRESS=$(nibid keys show $WALLET -a)
NIBIRU_VALOPER_ADDRESS=$(nibid keys show $WALLET --bech val -a)
echo 'export NIBIRU_WALLET_ADDRESS='${NIBIRU_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export NIBIRU_VALOPER_ADDRESS='${NIBIRU_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Faucet

```bash
curl -X POST -d '{"address": "'"$NIBIRU_WALLET_ADDRESS"'", "coins": ["10000000unibi","100000000000unusd"]}' https://faucet.testnet-2.nibiru.fi/
```
### To check your wallet balance:
```bash
nibid query bank balances $NIBIRU_WALLET_ADDRESS
```

### Create validator
After your node is synced, create validator
```bash
nibid tx staking create-validator \
  --amount 10000000unibi \
  --from $WALLET \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(nibid tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $NIBIRU_CHAIN_ID \
  --fees=5000unibi -y
```

## Usefull commands
### Service management
Check logs
```bash
journalctl -fu nibid -o cat
```

Start service
```bash
sudo systemctl start nibid
```

Stop service
```bash
sudo systemctl stop nibid
```

Restart service
```bash
sudo systemctl restart nibid
```

### Node info
Synchronization info
```bash
nibid status 2>&1 | jq .SyncInfo
```

Validator info
```bash
nibid status 2>&1 | jq .ValidatorInfo
```

Node info
```bash
nibid status 2>&1 | jq .NodeInfo
```

Show node id
```bash
nibid tendermint show-node-id
```

### Wallet operations
List of wallets
```bash
nibid keys list
```

Recover wallet
```bash
nibid keys add $WALLET --recover
```

Delete wallet
```bash
nibid keys delete $WALLET
```

Get wallet balance
```bash
nibid query bank balances $NIBIRU_WALLET_ADDRESS
```

Transfer funds
```bash
nibid tx bank send $NIBIRU_WALLET_ADDRESS <TO_NIBIRU_WALLET_ADDRESS> 10000000unibi --fees=5000unibi
```

### Voting
```bash
nibid tx gov vote 1 yes --from $WALLET --chain-id=$NIBIRU_CHAIN_ID --fees=5000unibi
```

### Staking, Delegation and Rewards
Delegate stake
```bash
nibid tx staking delegate $NIBIRU_VALOPER_ADDRESS 10000000unibi --from=$WALLET --chain-id=$NIBIRU_CHAIN_ID --fees=5000unibi
```

Redelegate stake from validator to another validator
```bash
nibid tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000unibi --from=$WALLET --chain-id=$NIBIRU_CHAIN_ID --fees=5000unibi
```

Withdraw all rewards
```bash
nibid tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$NIBIRU_CHAIN_ID --fees=5000unibi
```

Withdraw rewards with commision
```bash
nibid tx distribution withdraw-rewards $NIBIRU_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$NIBIRU_CHAIN_ID --fees=5000unibi
```

### Validator management
Edit validator
```bash
nibid tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$NIBIRU_CHAIN_ID \
  --from=$WALLET
```

Unjail validator
```bash
nibid tx slashing unjail \
  --from=$WALLET \
  --chain-id=$NIBIRU_CHAIN_ID \
  --fees=5000unibi -y
```

### Delete node
```bash
sudo systemctl stop nibid
sudo systemctl disable nibid
sudo rm /etc/systemd/system/nibi* -rf
sudo rm $(which nibid) -rf
sudo rm $HOME/.nibid* -rf
sudo rm $HOME/nibiru -rf
sed -i '/NIBIRU_/d' ~/.bash_profile
```
