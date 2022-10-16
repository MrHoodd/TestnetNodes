
# Official Links
### [Official Document](https://github.com/mande-labs/testnet-1)
### [Discord](https://discord.gg/Q43H94fG7X)

# Explorer
[Explorer](https://explorer.stavr.tech/mande-chain/staking)

# Install Node Guide

### Setting up variables
Specify the name of your moniker (validator) which will be visible in the explorer
```bash
NODENAME=<YOUR_MONIKER_NAME>
```
### Save and import variables into system
```bash
MANDE_PORT=30
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export MANDE_CHAIN_ID=mande-testnet-1" >> $HOME/.bash_profile
echo "export MANDE_PORT=${MANDE_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Update Packages and Depencies
```bash
sudo apt update && sudo apt upgrade -y
```

### Install Depencies
```bash
sudo apt install curl tar wget tmux htop net-tools clang pkg-config libssl-dev jq build-essential git make ncdu -y
```

### Install GO
```bash
ver="1.19" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

### Download binaries
```bash
cd $HOME
curl -OL https://github.com/mande-labs/testnet-1/raw/main/mande-chaind
mv mande-chaind /usr/local/bin
chmod 744 /usr/local/bin/mande-chaind
```

### Config app
```bash
mande-chaind config chain-id $MANDE_CHAIN_ID
mande-chaind config keyring-backend test
mande-chaind config node tcp://localhost:${MANDE_PORT}657
```

### Init app
```bash
mande-chaind init $NODENAME --chain-id MANDE_CHAIN_ID
```

### Download genesis file
```bash
wget -O $HOME/.mande-chain/config/genesis.json "https://raw.githubusercontent.com/mande-labs/testnet-1/main/genesis.json"
```

### Download addrbook
```bash
wget -O $HOME/.mande-chain/config/addrbook.json https://github.com/MrHoodd/TestnetNodes/blob/main/MandeChain/addrbook.json
```

### Set minimum gas price, seeds, and peers
```bash
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0mand\"/;" ~/.mande-chain/config/app.toml
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.mande-chain/config/config.toml
peers="cd3e4f5b7f5680bbd86a96b38bc122aa46668399@34.171.132.212:26656,6780b2648bd2eb6adca2ca92a03a25b216d4f36b@34.170.16.69:26656,a3e3e20528604b26b792055be84e3fd4de70533b@38.242.199.93:24656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.mande-chain/config/config.toml
seeds="cd3e4f5b7f5680bbd86a96b38bc122aa46668399@34.171.132.212:26656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.mande-chain/config/config.toml
```

## Set custom ports
```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${MANDE_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${MANDE_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${MANDE_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${MANDE_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${MANDE_PORT}660\"%" $HOME/.noisd/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${MANDE_PORT}317\"%; s%^address = \":8080\"%address = \":${MANDE_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${MANDE_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${MANDE_PORT}091\"%" $HOME/.noisd/config/app.toml
```

### Config pruning (Optional)
```bash
pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.mande-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.mande-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.mande-chain/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.mande-chain/config/app.toml
```

### Indexer (Optional)
```bash
indexer="null" && \
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.mande-chain/config/config.toml
```

### Enable prometheus
```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.mande-chain/config/config.toml
```

### Reset chain data
```bash
mande-chaind tendermint unsafe-reset-all --home $HOME/.mande-chain
```

### Create service
```bash
sudo tee /etc/systemd/system/mande-chaind.service > /dev/null <<EOF
[Unit]
Description=mande-chaind
After=network-online.target

[Service]
User=$USER
ExecStart=$(which mande-chaind) start
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
sudo systemctl enable mande-chaind
sudo systemctl restart mande-chaind && sudo journalctl -u mande-chaind -f -o cat
```

### State-Sync (Optional)
```bash
SNAP_RPC=http://38.242.199.93:24657
peers="a3e3e20528604b26b792055be84e3fd4de70533b@38.242.199.93:24656"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.mande-chain/config/config.toml
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 500)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.mande-chain/config/config.toml

mande-chaind tendermint unsafe-reset-all --home /root/.mande-chain --keep-addr-book
systemctl restart mande-chaind && journalctl -u mande-chaind -f -o cat
```

### Create wallet
To create a new wallet, don't forget to save the mnemonics 
```bash
mande-chaind keys add $WALLET
```

To recover existing keys use
```bash
mande-chaind keys add $WALLET --recover
```

List of wallets
```bash
mande-chaind keys list
```
### Save wallet info
Add wallet and valoper address into variables 
```bash
MANDE_WALLET_ADDRESS=$(mande-chaind keys show $WALLET -a)
MANDE_VALOPER_ADDRESS=$(mande-chaind keys show $WALLET --bech val -a)
echo 'export MANDE_WALLET_ADDRESS='${MANDE_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export MANDE_VALOPER_ADDRESS='${MANDE_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Ask for faucet
```bash
curl -d '{"address":"<MANDE ADDRESS>"}' -H 'Content-Type: application/json' http://35.224.207.121:8080/request
```
### To check your wallet balance:
```bash
mande-chaind query bank balances $MANDE_WALLET_ADDRESS
```

### Create validator
After your node is synced, create validator

To check if your node is synced simply run
```bash
curl http://localhost:26657/status sync_info "catching_up": false
```
To create your validator run command below
```bash
mande-chaind tx staking create-validator \
--chain-id $MANDE_CHAIN_ID \
--amount 100000000cred \
--pubkey "$(mande-chaind tendermint show-validator)" \
--from $WALLET \
--moniker $NODENAME \
--identity=<your_keybase_id> \
--website="<your_website>" \
--details="<your_validator_description>" \
--fees 1000mand
```
 
## Usefull commands
### Service management
Check logs
```bash
journalctl -fu mande-chaind -o cat
```

Start service
```bash
sudo systemctl start mande-chaind
```

Stop service
```bash
sudo systemctl stop mande-chaind
```

Restart service
```bash
sudo systemctl restart mande-chaind
```

### Node info
Synchronization info
```bash
mande-chaind status 2>&1 | jq .SyncInfo
```

Validator info
```bash
mande-chaind status 2>&1 | jq .ValidatorInfo
```

Node info
```bash
mande-chaind status 2>&1 | jq .NodeInfo
```

Show node id
```bash
mande-chaind tendermint show-node-id
```

### Wallet operations
List of wallets
```bash
mande-chaind keys list
```

Recover wallet
```bash
mande-chaind keys add $WALLET --recover
```

Delete wallet
```bash
mande-chaind keys delete $WALLET
```

Get wallet balance
```bash
mande-chaind query bank balances $MANDE_WALLET_ADDRESS
```

Transfer funds
```bash
mande-chaind tx bank send $MANDE_WALLET_ADDRESS <TO_NOIS_WALLET_ADDRESS> 10000000mand
```

### Voting
```bash
mande-chaind tx gov vote 1 yes --from $WALLET --chain-id=$MANDE_CHAIN_ID
```

### Staking, Delegation and Rewards
Delegate stake
```bash
mande-chaind tx staking delegate $MANDE_VALOPER_ADDRESS 10000000mand --from=$WALLET --chain-id=$MANDE_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```bash
mande-chaind tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000mand --from=$WALLET --chain-id=$MANDE_CHAIN_ID --gas=auto
```

Withdraw all rewards
```bash
mande-chaind tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$MANDE_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```bash
mande-chaind tx distribution withdraw-rewards $MANDE_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$MANDE_CHAIN_ID
```

### Validator management
Edit validator
```bash
mande-chaind tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$MANDE_CHAIN_ID \
  --from=$WALLET
```

Unjail validator
```bash
mande-chaind tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$MANDE_CHAIN_ID \
  --gas=auto
```
Delete node
```bash
sudo systemctl stop mande-chaind && \
sudo systemctl disable mande-chaind && \
rm /etc/systemd/system/mande-chaind.service && \
sudo systemctl daemon-reload && \
cd $HOME && \
rm -rf .mande-chain && \
rm -rf $(which mande-chaind)
```
