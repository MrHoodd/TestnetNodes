# Official Links
[Website](https://okp4.network/) [Twitter](https://twitter.com/OKP4_Protocol) [Discord](https://discord.gg/7EA28BJR)

# Explorer
[Explorer](https://explorer.stavr.tech/okp4/staking) [Explorer](https://okp4.explorers.guru/validators)

# Install Node Guide

### Create variables
Specify the name of your moniker (validator) which will be visible in the explorer OKP_NODENAME=<YOUR_MONIKER_NAME>
```bash
OKP_NODENAME=<YOUR_MONIKER_NAME>
OKP_WALLET=wallet
OKP=okp4d
OKP_ID=okp4-nemeton
OKP_PORT=45
OKP_FOLDER=.okp4d
OKP_FOLDER2=okp4d
OKP_VER=
OKP_REPO=https://github.com/okp4/okp4d.git
OKP_GENESIS=https://raw.githubusercontent.com/okp4/networks/main/chains/nemeton/genesis.json
OKP_ADDRBOOK=
OKP_MIN_GAS=0
OKP_DENOM=uknow
OKP_SEEDS=a7f1dcf7441761b0e0e1f8c6fdc79d3904c22c01@38.242.150.63:36656,2f9e54645aca860f703e3f756fa7c472b829a9a9@tenderseed.ccvalidators.com:26009
OKP_PEERS=f595a1386d5ca2e0d2cd81d3c6372c3bf84bbd16@65.109.31.114:2280,a49302f8999e5a953ebae431c4dde93479e17155@162.19.71.91:26656,b8330b2cb0b6d6d8751341753386afce9472bac7@89.163.208.12:26656,79d179ea2e1fbdcc0c59a95ab7f1a0c48438a693@65.108.106.131:26706,501ad80236a5ac0d37aafa934c6ec69554ce7205@89.149.218.20:26656,5fbddca54548bf125ee96bb388610fe1206f087f@51.159.66.123:26656,769f74d3bb149216d0ab771d7767bd39585bc027@185.196.21.99:26656,024a57c0bb6d868186b6f627773bf427ec441ab5@65.108.2.41:36656,fff0a8c202befd9459ff93783a0e7756da305fe3@38.242.150.63:16656,2bfd405e8f0f176428e2127f98b5ec53164ae1f0@142.132.149.118:26656,bf5802cfd8688e84ac9a8358a090e99b5b769047@135.181.176.109:53656,dc9a10f2589dd9cb37918ba561e6280a3ba81b76@54.244.24.231:26656,085cf43f463fe477e6198da0108b0ab08c70c8ab@65.108.75.237:6040,803422dc38606dd62017d433e4cbbd65edd6089d@51.15.143.254:26656,dc14197ed45e84ca3afb5428eb04ea3097894d69@88.99.143.105:26656,c0864edb1e36c52dbee47ce38d8b47ec364a9eb9@135.181.24.128:28656,43930c7e1cdcfeadf02b9705aefd9a0a59adc353@148.251.69.216:26656,2e877dac234099023a9237eb2e5a05cfb3893633@144.76.45.59:16656,efc552f1211516d578543fc56afcbfbb77c656bd@5.161.145.101:36656,1b0afc2af49098b5bf6e3c89d7d29ef336c47260@144.76.27.79:60756,de245278be4c3540f0a6a867c4bac83155b4ebac@178.62.30.239:46656
```

### Import variables into system
```bash
echo 'export OKP_NODENAME='$OKP_NODENAME >> $HOME/.bash_profile
echo "export OKP_WALLET=${OKP_WALLET}" >> $HOME/.bash_profile
echo "export OKP=${OKP}" >> $HOME/.bash_profile
echo "export OKP_ID=${OKP_ID}" >> $HOME/.bash_profile
echo "export OKP_PORT=${OKP_PORT}" >> $HOME/.bash_profile
echo "export OKP_FOLDER=${OKP_FOLDER}" >> $HOME/.bash_profile
echo "export OKP_FOLDER2=${OKP_FOLDER2}" >> $HOME/.bash_profile
echo "export OKP_VER=${OKP_VER}" >> $HOME/.bash_profile
echo "export OKP_REPO=${OKP_REPO}" >> $HOME/.bash_profile
echo "export OKP_GENESIS=${OKP_GENESIS}" >> $HOME/.bash_profile
echo "export OKP_PEERS=${OKP_PEERS}" >> $HOME/.bash_profile
echo "export OKP_SEED=${OKP_SEED}" >> $HOME/.bash_profile
echo "export OKP_MIN_GAS=${OKP_MIN_GAS}" >> $HOME/.bash_profile
echo "export OKP_DENOM=${OKP_DENOM}" >> $HOME/.bash_profile
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
cd $HOME
wget -O go1.18.4.linux-amd64.tar.gz https://golang.org/dl/go1.18.4.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.4.linux-amd64.tar.gz && rm go1.18.4.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
go version
```

### Download binaries
```bash
cd $HOME
git clone $OKP_REPO
cd $OKP_FOLDER2
make install
cp $HOME/go/bin/okp4d /usr/bin
```

### Config app
```bash
$OKP config chain-id $OKP_ID
$OKP config keyring-backend file
$OKP init $OKP_NODENAME --chain-id $OKP_ID
```

### Download genesis and addrbook
```bash
wget $OKP_GENESIS -O $HOME/$OKP_FOLDER/config/genesis.json
wget $OKP_ADDRBOOK -O $HOME/$OKP_FOLDER/config/addrbook.json
```

### Set seeds and peers
```bash
SEEDS="$OKP_SEEDS"
PEERS="$OKP_PEERS"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/$OKP_FOLDER/config/config.toml
```

### Config pruning (Optional)
```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/$OKP_FOLDER/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/$OKP_FOLDER/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/$OKP_FOLDER/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/$OKP_FOLDER/config/app.toml
```

### Set custom ports
```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:2${OKP_PORT}8\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:2${OKP_PORT}7\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${OKP_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:2${OKP_PORT}6\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":2${OKP_PORT}0\"%" $HOME/$OKP_FOLDER/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${OKP_PORT}7\"%; s%^address = \":8080\"%address = \":${OKP_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${OKP_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${OKP_PORT}91\"%" $HOME/$OKP_FOLDER/config/app.toml
sed -i.bak -e "s%^node = \"tcp://localhost:26657\"%node = \"tcp://localhost:2${OKP_PORT}7\"%" $HOME/$OKP_FOLDER/config/client.toml
```

### Enable prometheus
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/$OKP_FOLDER/config/config.toml
```

### Set minimum gas price
```bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.00125$OKP_DENOM\"/" $HOME/$OKP_FOLDER/config/app.toml
```

### Indexer (Optional)
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/$OKP_FOLDER/config/config.toml
```

### Reset chain data
```bash
$OKP tendermint unsafe-reset-all --home $HOME/$OKP_FOLDER
```

### Create service
```bash
sudo tee /etc/systemd/system/$OKP.service > /dev/null <<EOF
[Unit]
Description=$OKP
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which $OKP) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

### Register and start service
```bash
sudo systemctl daemon-reload
sudo systemctl enable $OKP
sudo systemctl restart $OKP
source $HOME/.bash_profile
```

### Create wallet
To create a new wallet, don't forget to save the mnemonics
```bash
okp4d keys add $OKP_WALLET
```
To recover existing keys use
```bash
okp4d keys add $OKP_WALLET --recover
```
List of wallets
```bash
okp4d keys list
```
###vSave wallet info
Add wallet and valoper address into variables
```bash
OKP_WALLET_ADDRESS=$(okp4d keys show $OKP_WALLET -a)
OKP_VALOPER_ADDRESS=$(okp4d keys show $OKP_WALLET --bech val -a)
echo 'export OKP_WALLET_ADDRESS='${OKP_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export OKP_VALOPER_ADDRESS='${OKP_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
### Ask for faucet

## [Faucet](https://faucet.okp4.network/)

### To check your wallet balance:
```bash
okp4d query bank balances $OKP_WALLET_ADDRESS
```
### Create validator
After your node is synced, create validator
To check if your node is synced simply run
```bash
okp4d status 2>&1 | jq .SyncInfo
```
To create your validator run command below
```bash
okp4d tx staking create-validator \
  --amount 1999000uknow \
  --from $OKP_WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(okp4d tendermint show-validator) \
  --moniker $OKP_NODENAME \
  --chain-id $OKP_ID \
  --fees 250uknow
```

## Usefull commands
### Service management
Check logs
```bash
journalctl -fu okp4d -o cat
```

Start service
```bash
systemctl start okp4d
```

Stop service
```bash
systemctl stop okp4d
```

Restart service
```bash
systemctl restart okp4d
```

## Node info
Synchronization info
```bash
okp4d status 2>&1 | jq .SyncInfo
```

Validator info
```bash
okp4d status 2>&1 | jq .ValidatorInfo
```

Node info
```bash
okp4d status 2>&1 | jq .NodeInfo
```

Show node id
```bash
okp4d tendermint show-node-id
```

## Wallet operations
List of wallets
```bash
okp4d keys list
```

Recover wallet
```bash
okp4d keys add $OKP_WALLET --recover
```

Delete wallet
```bash
okp4d keys delete $OKP_WALLET
```

Get wallet balance
```bash
okp4d query bank balances $OKP_WALLET_ADDRESS
```

Transfer funds
```bash
okp4d tx bank send $OKP_WALLET_ADDRESS <TO_WALLET_ADDRESS> 10000000uknow
```

### Voting
```bash
okp4d tx gov vote 1 yes --from $OKP_WALLET --chain-id=$OKP_ID
```

### Staking, Delegation and Rewards
Delegate stake
```bash
okp4d tx staking delegate $OKP_VALOPER_ADDRESS 10000000uknow --from=$OKP_WALLET --chain-id=$OKP_ID --gas=auto --fees 250uknow
```

Redelegate stake from validator to another validator
```bash
okp4d tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000uknow --from=$OKP_WALLET --chain-id=$OKP_ID --gas=auto --fees 250uknow
```

Withdraw all rewards
```bash
okp4d tx distribution withdraw-all-rewards --from=$OKP_WALLET --chain-id=$OKP_ID --gas=auto --fees 250uknow
```

Withdraw rewards with commision
```bash
okp4d tx distribution withdraw-rewards $OKP_VALOPER_ADDRESS --from=$OKP_WALLET --commission --chain-id=$OKP_ID
```

### Validator management
Edit validator
```bash
okp4d tx staking edit-validator \
  --moniker=$OKP_NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$OKP_ID \
  --from=$OKP_WALLET
```

Unjail validator
```bash
okp4d tx slashing unjail \
  --broadcast-mode=block \
  --from=$OKP_WALLET \
  --chain-id=$OKP_ID \
  --gas=auto \
  --fees 250uknow
```
Delete node
```bash
sudo systemctl stop okp4d
sudo systemctl disable okp4d
sudo rm /etc/systemd/system/okp4* -rf
sudo rm $(which okp4d) -rf
sudo rm $HOME/.okp4d* -rf
sudo rm $HOME/okp4d -rf
sed -i '/OKP_/d' ~/.bash_profile
```
