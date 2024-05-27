## Managing keys

Generate new key
```
nibid keys add wallet
```
Recover key
```
nibid keys add wallet --recover
```
List all key
```
nibid keys list
```
Delete key
```
nibid keys delete wallet
```
Export key
```
nibid keys export wallet
```
Import key
```
nibid keys import wallet wallet.backup
```
Query wallet balances
```
nibid q bank balances $(nibid keys show wallet -a)
```
## Managing validators 

Create validator
```
nibid tx staking create-validator \
--amount 1000000unibi \
--pubkey $(nibid tendermint show-validator) \
--moniker "your-moniker-name" \
--identity "your-keybase-id" \
--details "your-details" \
--website "your-website" \
--security-contact "your-email" \
--chain-id cataclysm-1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--fees 5000unibi \
-y
```

Backup Validator
```
cat $HOME/.nibid/config/priv_validator_key.json
```
Edit validator
```
nibid tx staking edit-validator \
--new-moniker "your-moniker-name" \
--identity "your-keybase-id" \
--details "your-details" \
--website "your-website" \
--security-contact "your-email" \
--chain-id cataclysm-1 \
--commission-rate 0.05 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--fees 5000unibi \
-y
```
Unjail validator
```
nibid tx slashing unjail --from wallet --chain-id cataclysm-1 --gas-adjustment 1.4 --gas auto --fees 5000unibi -y
```
View validator details
```
nibid q staking validator $(nibid keys show wallet --bech val -a)
```
Validator jail reason
```
nibid query slashing signing-info $(nibid tendermint show-validator)
```
List active validator
```
nibid q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
List incative validator
```
nibid q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
View validator details
```
nibid q staking validator $(nibid keys show wallet --bech val -a)
```
## Managing Tokens
Withdraw reward from all validator
```
nibid tx distribution withdraw-all-rewards --from wallet --chain-id cataclysm-1 --gas-adjustment 1.4 --gas auto --fees 5000unibi -y
```
Withdraw reward and commission
```
nibid tx distribution withdraw-rewards $(nibid keys show wallet --bech val -a) --commission --from wallet --chain-id cataclysm-1 --gas-adjustment 1.4 --gas auto --fees 5000unibi -y
```
Delegate tokens to your validator
```
nibid tx staking delegate <to-valoper-address> 1000000unibi --from wallet --chain-id cataclysm-1 --gas-adjustment 1.4 --gas auto --fees 5000unibi -y
```
Redelegate to another validator
```
nibid tx staking redelegate $(nibid keys show wallet --bech val -a) <to-valoper-address> 1000000unibi --from wallet --chain-id cataclysm-1 --gas-adjustment 1.4 --gas auto --fees 5000unibi -y
```
Unbond token from your own validator
```
nibid tx staking unbond $(nibid keys show wallet --bech val -a) 1000000unibi --from wallet --chain-id cataclysm-1 --gas-adjustment 1.4 --gas auto --fees 5000unibi -y
```
Send token to the wallet
```
nibid tx bank send wallet <to-wallet-address> 1000000unibi --from wallet --chain-id cataclysm-1 --gas-adjustment 1.4 --gas auto --fees 5000unibi -y
```
## Governance
Query list proposal
```
nibid query gov proposals
```
View proposal by ID
```
nibid query gov proposal 1
```
Vote option yes
```
nibid tx gov vote 1 yes --from wallet --chain-id cataclysm-1 --gas-adjustment 1.4 --gas auto --fees 5000unibi -y
```
Vote option no
```
nibid tx gov vote 1 no --from wallet --chain-id cataclysm-1 --gas-adjustment 1.4 --gas auto --fees 5000unibi -y
```
Vote option asbtain
```
nibid tx gov vote 1 abstain --from wallet --chain-id cataclysm-1 --gas-adjustment 1.4 --gas auto --fees 5000unibi -y
```
Vote option NoWithVeto
```
nibid tx gov vote 1 NoWithVeto --from wallet --chain-id cataclysm-1 --gas-adjustment 1.4 --gas auto --fees 5000unibi -y
```
## Usefulness
Update custom port
```
PORT=198
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}66\"%" $HOME/.nibid/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}17\"%; s%^address = \":8080\"%address = \":${PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}91\"%" $HOME/.nibid/config/app.toml
```
Disable indexer
```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.nibid/config/config.toml
```
Enable indexer
```
sed -i -e 's|^indexer *=.*|indexer = "kv"|' $HOME/.nibid/config/config.toml
```
Pruning update
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.nibid/config/app.toml
```
## Maintenance
Get validator information
```
nibid status 2>&1 | jq .ValidatorInfo
```
Get sync information
```
nibid status 2>&1 | jq .SyncInfo
```
Get node peer
```
echo $(nibid tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.nibid/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```
Check validator keys
```
[[ $(nibid q staking validator $(nibid keys show wallet --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(nibid status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```
Get live peers
```
curl -sS http://localhost:11857/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```
Configure minimum gas prices
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025unibi\"/" $HOME/.nibid/config/app.toml
```
Enable prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.nibid/config/config.toml
```
Reset chain data
```
nibid tendermint unsafe-reset-all --keep-addr-book --home $HOME/.nibid --keep-addr-book
```
## Remove node
```
cd $HOME
sudo systemctl stop nibiru
sudo systemctl disable nibiru
sudo rm /etc/systemd/system/nibiru.service
sudo systemctl daemon-reload
sudo rm -f $(which nibid)
sudo rm -rf $HOME/.nibid
sudo rm -rf $HOME/nibiru
sudo rm -rf $HOME/go
```