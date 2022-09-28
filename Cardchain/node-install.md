# Node setup

## Setting up vars
Here you have to put name of your moniker (validator) that will be visible in explorer
```
NODENAME=<YOUR_MONIKER_NAME_GOES_HERE>
```

Save and import variables into system
```
CARDCHAIN_PORT=100
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export CARDCHAIN_CHAIN_ID=Cardchain" >> $HOME/.bash_profile
echo "export CARDCHAIN_PORT=${CARDCHAIN_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Update packages
```
sudo apt update && sudo apt upgrade -y
```

## Install dependencies
```
sudo apt install curl build-essential git wget jq make gcc tmux chrony -y
```

## Install go
```
if ! [ -x "$(command -v go)" ]; then
  ver="1.18.2"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

## Download and build binaries
```
cd $HOME
curl https://get.ignite.com/DecentralCardGame/Cardchain@latest! | sudo bash
```

## Init app
```
Cardchain init $NODENAME --chain-id $CARDCHAIN_CHAIN_ID
```

## Download genesis and addrbook
```
wget -O $HOME/.Cardchain/config/genesis.json "https://raw.githubusercontent.com/DecentralCardGame/Testnet1/main/genesis.json"
```

## Set seeds and peers
```
SEEDS=""
PEERS="a89083b131893ca8a379c9b18028e26fa250473c@159.69.11.174:36656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.Cardchain/config/config.toml
```

## Set custom ports
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CARDCHAIN_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CARDCHAIN_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CARDCHAIN_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CARDCHAIN_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CARDCHAIN_PORT}660\"%" $HOME/.Cardchain/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CARDCHAIN_PORT}317\"%; s%^address = \":8080\"%address = \":${CARDCHAIN_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CARDCHAIN_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CARDCHAIN_PORT}091\"%" $HOME/.Cardchain/config/app.toml
```

## Config pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.Cardchain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.Cardchain/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.Cardchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.Cardchain/config/app.toml
```

## Set minimum gas price and timeout commit
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0ubpf\"/" $HOME/.Cardchain/config/app.toml
```
## Set bad peers filter (optional)
```
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $HOME/.Cardchain/config/config.toml
```
## Enable snapshots (optional)
```
snapshot_interval=1000 && \ sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.Cardchain/config/app.toml```
```
## Enable prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.Cardchain/config/config.toml
```

## Reset chain data
```
Cardchain tendermint unsafe-reset-all --home $HOME/.Cardchain
```

## Create service
```
sudo tee /etc/systemd/system/Cardchain.service > /dev/null <<EOF
[Unit]
Description=Cardchain
After=network-online.target

[Service]
User=$USER
ExecStart=$(which Cardchain) start --home $HOME/.Cardchain
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Register and start service
```
sudo systemctl daemon-reload
sudo systemctl enable Cardchain
sudo systemctl restart Cardchain && sudo journalctl -u Cardchain -f -o cat
```
