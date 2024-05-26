### Minimum hardware requirement
8 Cores, 16G Ram,  500G SSD, Ubuntu 22.04

Website: https://nibiru.fi <br>
Docs: https://nibiru.fi/docs/<br>
Telegram: https://t.me/nibiruchain <br>
Explores: https://explorer.nibiru.fi/cataclysm-1 <br>
<br>
### Update system & Install packages
```
sudo apt update && sudo apt upgrade -y
sudo apt install make curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```
### Install GO
```
ver="1.21.5"
wget "https://golang.org/dl/go$ver.linux-arm64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-arm64.tar.gz"
rm "go$ver.linux-arm64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```
### Download and build binaries
```
cd $HOME
rm -rf nibiru
git clone https://github.com/NibiruChain/nibiru.git
cd nibiru
git checkout v1.3.0
make install
```
### Node configuration
```
nibid init Moniker --chain-id=cataclysm-1
```
```
curl -Ls https://snapshots.kjnodes.com/nibiru/genesis.json > $HOME/.nibid/config/genesis.json
curl -Ls https://snapshots.kjnodes.com/nibiru/addrbook.json > $HOME/.nibid/config/addrbook.json
```
### Add seeds
```
NETWORK=cataclysm-1
sed -i 's|\<persistent_peers\> =.*|persistent_peers = "'$(curl -s https://networks.nibiru.fi/$NETWORK/peers)'"|g' $HOME/.nibid/config/config.toml
```

### Set minimum gas price
```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.025unibi\"|" $HOME/.nibid/config/app.toml
```
### Set pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.nibid/config/app.toml
```

#### Download latest chain snapshot
```
curl -L https://snapshots.kjnodes.com/nibiru/snapshot_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.nibid
```
### Create Service and Start node
```
sudo tee /etc/systemd/system/nibid.service > /dev/null <<EOF
[Unit]
Description=Nibi Validator
After=network-online.target
[Service]
User=$USER
ExecStart=$(which nibid) start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
```
sudo systemctl daemon-reload
sudo systemctl enable nibid
sudo systemctl restart nibid
sudo journalctl -u nibid -f --no-hostname -o cat
```

### Add New Wallet Key - Save seed
```
nibid keys add walletWallet
```
### Recover existing key
```
nibid keys add wallet --recover
```
### List All Keys
```
nibid keys list
```
### Query Wallet Balance
```
nibid q bank balances $(nibid keys show wallet -a)
```
### Check sync status > False is synced
```
nibid status 2>&1 | jq .SyncInfo.catching_up
```
### Create Validator
Change your info "XXXXXX"
```
nibid tx staking create-validator \
--amount 1000000unibi \
--pubkey $(nibid tendermint show-validator) \
--moniker "XXXXXX" \
--identity "XXXXXX" \
--details "XXXXXX" \
--website "XXXXXX" \
--security-contact "XXXXXX" \
--chain-id cataclysm-1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.05 \
--min-self-delegation 1 \
--from wallet \
--fees 5000unibi \
-y
```
### Backup Validator > Important
```
cat $HOME/.nibid/config/priv_validator_key.json
```
### Edit Existing Validator > Change your info 
```
nibid tx staking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id cataclysm-1 \
--commission-rate 0.05 \
--from wallet \
--fees 5000unibi \
-y
```
### Delegate Token to your own validator
```
nibid tx staking delegate $(nibid keys show wallet --bech val -a) 1000000unibi --from wallet --chain-id cataclysm-1 --fees 5000unibi -y
```
### Withdraw rewards and commission from your validator
```
nibid tx distribution withdraw-rewards $(nibid keys show wallet --bech val -a) --commission --from wallet --chain-id cataclysm-1 --fees 5000unibi -y
```
### Unjail validator
```
nibid tx slashing unjail --from wallet --chain-id cataclysm-1 --fees 5000unibi -y
```
## Services Management
#### Reload Service
```
sudo systemctl daemon-reload
```
### Enable Service
```
sudo systemctl enable nibid
```
### Disable Service
```
sudo systemctl disable nibid
```
### Start Service
```
sudo systemctl start nibid
```
### Stop Service
```
sudo systemctl stop nibid
```
### Restart Service
```
sudo systemctl restart nibid
```
### Check Service Status
```
sudo systemctl status nibid
```
### Check Service Logs
```
sudo journalctl -u nibid -f --no-hostname -o cat
```
## Remove node
```
cd $HOME
sudo systemctl stop nibid.service
sudo systemctl disable nibid.service
sudo rm /etc/systemd/system/nibid.service
sudo systemctl daemon-reload
rm -f $(which nibid)
rm -rf $HOME/.nibid
rm -rf $HOME/nibiru
```



