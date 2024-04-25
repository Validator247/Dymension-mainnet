# Dymension-mainnet

Update system

    sudo apt update
    sudo apt-get install git curl build-essential make jq gcc snapd chrony lz4 tmux unzip bc -y

Install Go

    sudo rm -rvf /usr/local/go/
    wget https://golang.org/dl/go1.21.1.linux-amd64.tar.gz
    sudo tar -C /usr/local -xzf go1.21.1.linux-amd64.tar.gz
    rm go1.21.1.linux-amd64.tar.gz

Configure Go

    export GOROOT=/usr/local/go
    export GOPATH=$HOME/go
    export GO111MODULE=on
    export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin

Install Cosmovisor

    go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.0.0

Install Node

    git clone https://github.com/dymensionxyz/dymension.git dymension
    cd dymension
    git checkout v3.1.0
    make install

Configure Node

Initialize Node
Please replace YOUR_MONIKER with your own moniker:

    dymd init YOUR_MONIKER --chain-id dymension_1100-1

Download Genesis

    wget -O genesis.json https://snapshots.polkachu.com/genesis/dymension/genesis.json --inet4-only
    mv genesis.json ~/.dymension/config

Configure Seed

    sed -i 's/seeds = ""/seeds = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:20556"/' ~/.dymension/config/config.toml

Configure Cosmovisor Folder

    # Create Cosmovisor Folders
    mkdir -p ~/.dymension/cosmovisor/genesis/bin
    mkdir -p ~/.dymension/cosmovisor/upgrades

    # Load Node Binary into Cosmovisor Folder
    cp ~/go/bin/dymd ~/.dymension/cosmovisor/genesis/bin

Create Service File

    sudo tee /etc/systemd/system/dymd.service > /dev/null <<EOF
    [Unit]
    Description=dymd Daemon
    After=network-online.target
    [Service]
    User=$USER
    ExecStart=$(which dymd) start
    Restart=always
    RestartSec=3
    LimitNOFILE=65535
    [Install]
    WantedBy=multi-user.target
    EOF
    
    sudo systemctl daemon-reload
    sudo systemctl enable dymd

# Download Snapshot

Install lz4 if needed

    sudo apt update
    sudo apt install snapd -y
    sudo snap install lz4

Download the snapshot

    curl -o - -L https://snapshots.polkachu.com/snapshots/dymension/dymension_1170907.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.dymension

Launch Node

    sudo systemctl restart dymd
    journalctl -u dymd -f -o cat

# Wallet & Validator

Add new key

    dymd keys add wallet

Recover existing key

    dymd keys add wallet --recover

Create Validator

    dymd tx staking create-validator \
      --amount 1000000adym \
      --commission-max-change-rate "0.05" \
      --commission-max-rate "0.10" \
      --commission-rate "0.05" \
      --min-self-delegation "1" \
      --pubkey=$(dymd tendermint show-validator) \
      --moniker 'Your_nodename' \
      --website "Your_website" \
      --identity "your_keybase" \
      --details "I Love Validator247" \
      --security-contact "Your_mail" \
      --chain-id dymension_1100-1 \
      --gas auto --gas-prices 20000000000adym \
      --gas-adjustment 1.6 \
      --from wallet -y

# DONE               
