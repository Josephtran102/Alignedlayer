## Cấu hình đề xuất:
```py
- Memory: 16 GB RAM
- CPU: 4 cores
- Disk: 200GB of storage (NVME)
- Linux amd64 arm64 (The guide was tested on Ubuntu 22.04.4 LTS)
```
# Cài đặt dependencies:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```
# Cài đặt Go:
```bash
cd $HOME
VER="1.21.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

# Set vars
```bash
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export ALIGNEDLAYER_CHAIN_ID="alignedlayer"" >> $HOME/.bash_profile
echo "export ALIGNEDLAYER_PORT="50"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

# Download binary
```bash
cd $HOME
#install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source $HOME/.cargo/env
#install Ignite CLI
curl https://get.ignite.com/cli | bash
sudo mv ignite /usr/local/bin/
#install bin
rm -rf $HOME/aligned_layer_tendermint
git clone https://github.com/yetanotherco/aligned_layer_tendermint.git
cd $HOME/aligned_layer_tendermint
git checkout 98643167990f8a597b331ddd879e079bafb25b08
make build-linux
```

# config and init app
alignedlayerd init $MONIKER --chain-id alignedlayer
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${ALIGNEDLAYER_PORT}657\"|" $HOME/.alignedlayer/config/client.toml

# download genesis and addrbook
wget -O $HOME/.alignedlayer/config/genesis.json https://testnet-files.itrocket.net/alignedlayer/genesis.json
wget -O $HOME/.alignedlayer/config/addrbook.json https://testnet-files.itrocket.net/alignedlayer/addrbook.json

# set seeds and peers
SEEDS="d1a8816c1c5800b352c2a1eb0e7a156bce34ae9f@alignedlayer-testnet-seed.itrocket.net:50656"
PEERS="144c2d4fbbaf54dda837bfbc88b688fb2f02c92f@alignedlayer-testnet-peer.itrocket.net:50656,2567ea5aed4bba4e3062a1072a8f1e7fb4e4497c@65.109.85.36:26656,4093bf12076818a82f9fc1c75dc974e1d93daf44@195.201.30.159:26656,5af5d7438b4e34ae9672e01ca3293ad99afede0b@149.50.101.11:26656,692729135ab36bf8e9fbd65ce8f1913665bed299@188.40.109.171:26656,7292de855372480ae23dbcaf94d36ead187cf6a8@194.163.143.206:50656,a1d6d9569789a7a8765f0a4899439819f07755d4@213.133.103.213:26656,93ff29608b73176ce58f404d81338bc1b50d9701@173.212.241.186:26656,afeea4cd47aa80504adbdaa8aa019864e291de55@[2a03:cfc0:8000:13::b910:277f]:13356,51ca4087558ebe93a16e3f1e84a969d30e7a91f1@95.216.245.35:26656,8c9f5892a95058de7cd74224da42feaef0f1012f@84.247.143.162:5456"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.alignedlayer/config/config.toml

# set custom ports in app.toml
sed -i.bak -e "s%:1317%:${ALIGNEDLAYER_PORT}317%g;
s%:8080%:${ALIGNEDLAYER_PORT}080%g;
s%:9090%:${ALIGNEDLAYER_PORT}090%g;
s%:9091%:${ALIGNEDLAYER_PORT}091%g;
s%:8545%:${ALIGNEDLAYER_PORT}545%g;
s%:8546%:${ALIGNEDLAYER_PORT}546%g;
s%:6065%:${ALIGNEDLAYER_PORT}065%g" $HOME/.alignedlayer/config/app.toml

# set custom ports in config.toml file
sed -i.bak -e "s%:26658%:${ALIGNEDLAYER_PORT}658%g;
s%:26657%:${ALIGNEDLAYER_PORT}657%g;
s%:6060%:${ALIGNEDLAYER_PORT}060%g;
s%:26656%:${ALIGNEDLAYER_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ALIGNEDLAYER_PORT}656\"%;
s%:26660%:${ALIGNEDLAYER_PORT}660%g" $HOME/.alignedlayer/config/config.toml

# config pruning
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.alignedlayer/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.alignedlayer/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.alignedlayer/config/app.toml

# set minimum gas price, enable prometheus and disable indexing
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.0001stake"|g' $HOME/.alignedlayer/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.alignedlayer/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.alignedlayer/config/config.toml

# create service file
sudo tee /etc/systemd/system/alignedlayerd.service > /dev/null <<EOF
[Unit]
Description=Alignedlayer node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.alignedlayer
ExecStart=$(which alignedlayerd) start --home $HOME/.alignedlayer
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

# reset and download snapshot
alignedlayerd tendermint unsafe-reset-all --home $HOME/.alignedlayer
if curl -s --head curl https://testnet-files.itrocket.net/alignedlayer/snap_alignedlayer.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://testnet-files.itrocket.net/alignedlayer/snap_alignedlayer.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.alignedlayer
    else
  echo no have snap
fi

# enable and start service
sudo systemctl daemon-reload
sudo systemctl enable alignedlayerd
sudo systemctl restart alignedlayerd && sudo journalctl -u alignedlayerd -f

