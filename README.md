# Airchains

### Airchains node Installation Instructions.

[Official documentation](https://docs.airchains.io/)

System requirements:</br>
CPU: Quad Core or larger AMD or Intel (amd64) CPU
RAM:32GB RAM
SSD:1TB NVMe Storage
100MBps bidirectional internet connection
OS: Ubuntu 20.04 or 22.04</br>

You can take a weaker server

### Network configurations: </br>
Outbound - allow all traffic </br>
Inbound - open the following ports :</br>
1317 - REST </br>
26657 - TENDERMINT_RP </br>
26656 - Cosmos </br>
Running as a Provider? Add these specific ports: </br>
22221 - provider port </br>
22231 - provider port </br>
22241 - provider port </br>

### Installing the Babylon Node

1. Preparing the server/Required packages installation</br>
```
sudo apt update
sudo apt upgrade -y
sudo apt-get install libclang-dev
sudo apt install -y unzip logrotate git jq sed wget curl coreutils systemd
```
### Go installation.
```
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.21.6.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
source .bash_profile
go version
```

### Download and build binaries
```
cd $HOME && mkdir -p go/bin/
wget https://github.com/airchains-network/junction/releases/download/v0.1.0/junctiond
chmod +x junctiond
mv junctiond $HOME/go/bin/
```

# Config and init app
```
junctiond config set client chain-id junction
junctiond config set client keyring-backend test
junctiond config set client node tcp://localhost:26657
junctiond init "your moniker" --chain-id junction
```

# Download genesis and addrbook
```
curl -L https://snapshots-testnet.nodejumper.io/airchains-testnet/genesis.json > $HOME/.junction/config/genesis.json
curl -L https://snapshots-testnet.nodejumper.io/airchains-testnet/addrbook.json > $HOME/.junction/config/addrbook.json
```

# Set seeds and peers
```
sed -i -e 's|^seeds *=.*|seeds = "de2e7251667dee5de5eed98e54a58749fadd23d8@34.22.237.85:26656,48887cbb310bb854d7f9da8d5687cbfca02b9968@35.200.245.190:26656,2d1ea4833843cc1433e3c44e69e297f357d2d8bd@5.78.118.106:26656"|' $HOME/.junction/config/config.toml
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.001amf"|' $HOME/.junction/config/app.toml
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.junction/config/app.toml
```

# Create service file
```
sudo tee /etc/systemd/system/junctiond.service > /dev/null << EOF
[Unit]
Description=Airchains node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which junctiond) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable junctiond.service
```

# Reset and download snapshot
```
curl "https://snapshots-testnet.nodejumper.io/airchains-testnet/airchains-testnet_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.junction"
```

# enable and start service
```
sudo systemctl start junctiond.service
sudo journalctl -u junctiond.service -f --no-hostname -o cat
```

### Becoming a Validator

# Create wallet key new
```
junctiond keys add wallet
```

(OPTIONAL) RECOVER EXISTING KEY
```
junctiond keys add wallet --recover
```

# check sync status, once your node is fully synced, the output from above will print "false"
```
junctiond status 2>&1 | jq -r '.SyncInfo.catching_up // .sync_info.catching_up'
```

### We receive tokens from the tap in the [discord](https://discord.gg/airchains)
```
#switchyard-faucet-bot $faucet adress
```

# before creating a validator, you need to fund your wallet and check balance
```
junctiond q bank balances $(junctiond keys show wallet -a)
```
# Create validator
```
junctiond tx staking create-validator \
--amount=1000000amf \
--pubkey=$(junctiond tendermint show-validator) \
--moniker="Moniker" \
--identity=FFB0AA51A2DF5955 \
--details="I love YTWO❤️" \
--chain-id=junction \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.001amf \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```

### Update
```
No update

Current network:junction
Current version:v1.0.0
```

### Useful commands

Check balance
```
junctiond q bank balances $(junctiond keys show wallet -a)
```

CHECK SERVICE LOGS
```
sudo journalctl -u junctiond -f --no-hostname -o cat
```

RESTART SERVICE
```
sudo systemctl restart junctiond
```

GET VALIDATOR INFO
```
junctiond status 2>&1 | jq -r '.ValidatorInfo // .validator_info'
```

DELEGATE TOKENS TO YOURSELF
```
junctiond tx staking delegate $(junctiond keys show wallet --bech val -a) 1000000amf --from wallet --chain-id junction --gas-prices 0.001amf --gas-adjustment 1.5 --gas auto -y 
```

REMOVE NODE
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!
```
sudo systemctl stop junctiond && sudo systemctl disable junctiond && sudo rm /etc/systemd/system/junctiond.service && sudo systemctl daemon-reload && rm -rf $HOME/.junction && rm -rf junction && sudo rm -rf $(which junctiond) 
```
