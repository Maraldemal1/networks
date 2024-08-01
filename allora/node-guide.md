# Installation Guide

### Install Dependencies

```bash
# Update system package and install build tools

sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential fail2ban ufw
sudo apt -qy upgrade
```

### Configure Moniker

```bash
# Replace <your-moniker-name> with your own validator name

MONIKER="<your-moniker-name>"
```

### Install Go
```bash
sudo rm -rf /usr/local/go
curl -Ls https://go.dev/dl/go1.21.1.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
```


### Build Binaries
```bash
# Cloning project repository & Compile binaries

cd $HOME
git clone https://github.com/allora-network/allora-chain allora
cd alloragit
checkout v0.0.10
make build
```

```bash
# Prepare binaries for cosmovisor

mkdir -p $HOME/.allorad/cosmovisor/genesis/bin
mv build/allorad $HOME/.allorad/cosmovisor/genesis/bin/
rm -rf build
```

```bash
# Create symlinks

sudo ln -s $HOME/.allorad/cosmovisor/genesis $HOME/.allorad/cosmovisor/current -f
sudo ln -s $HOME/.allorad/cosmovisor/current/bin/allorad /usr/local/bin/allorad -f
```

### Cosmovisor Setup
```bash
# Install cosmovisor

go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```


### Create Service
```bash
# Create a systemd service

sudo tee /etc/systemd/system/allora.service > /dev/null << EOF
[Unit]
Description=allora node service
After=network-online.target
 
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.allorad"
Environment="DAEMON_NAME=allorad"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.allorad/cosmovisor/current/bin"
 
[Install]
WantedBy=multi-user.target
EOF
```

### Enable Service
```bash
# Enable allora systemd service

sudo systemctl daemon-reload
sudo systemctl enable allora
```

### Initialize Node
```bash
# Setting node configuration

allorad config set client chain-id testnet
allorad config set client keyring-backend test
allorad config set client node tcp://localhost:24657
```
```bash
# Initialize node

allorad init $MONIKER --chain-id testnet
```

### Download Genesis & Addrbook
```bash
# Download genesis & addrbook file

curl -Ls https://snap.maraldemal1.xyz/allora-testnet/genesis.json > $HOME/.allorad/config/genesis.json
curl -Ls https://snap.maraldemal1.xyz/allora-testnet/addrbook.json > $HOME/.allorad/config/addrbook.json
```

### Configure Seeds
```bash
# Setting up a seed peers

sed -i -e "s|^seeds *=.*|seeds = \"d1d43cc7c7aef715957289fd96a114ecaa7ba756@testnet-seeds.maraldemal1.xyz:24610\"|" $HOME/.allorad/config/config.toml
```

### Configure Gas Prices
```bash
# Setting up a gas prices

sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0uallo\"|" $HOME/.allorad/config/app.toml
```

### Pruning Setting
```bash
# Configure pruning setting

sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.allorad/config/app.toml
```

### Custom Port (Optional)
```bash
# Configure a custom port

sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:24658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:24657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:24660\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:24656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":24666\"%" $HOME/.allorad/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:24617\"%; s%^address = \":8080\"%address = \":24680\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:24690\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:24691\"%; s%:8545%:24645%; s%:8546%:24646%; s%:6065%:24665%" $HOME/.allorad/config/app.toml
```

### Download Snapshots
```bash
# Download latest chain snapshot

curl -L https://snap.maraldemal1.xyz/allora-testnet/allora-latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.allorad
[[ -f $HOME/.allorad/data/upgrade-info.json ]] && cp $HOME/.allorad/data/upgrade-info.json $HOME/.allorad/cosmovisor/genesis/upgrade-info.json
```

### Start Service
```bash
sudo systemctl start allora
```
