## Cấu hình đề xuất:
```py
- Memory: 16 GB RAM
- CPU: 4 cores
- Disk: 200GB of storage (NVME)
- Linux amd64 arm64 (The guide was tested on Ubuntu 22.04.4 LTS)
```
#### Software:
- [Go v1.22](https://go.dev/dl/)
- [jq](https://jqlang.github.io/jq/download/)
- [sponge](https://linux.die.net/man/1/sponge)
- [make](https://www.gnu.org/software/make/#download)
- [gcc](https://gcc.gnu.org/install/)

# 1. Cài đặt dependencies:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```
# 2. Cài đặt Go:
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
# 3. Set vars: (Edit tên ví, tên node MOIKER hoặc để mặc định có thể edit sau)
```bash
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export ALIGNEDLAYER_CHAIN_ID="alignedlayer"" >> $HOME/.bash_profile
echo "export ALIGNEDLAYER_PORT="50"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
# 4. Download binary
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
# 5. Config and init app:
```bash
alignedlayerd init $MONIKER --chain-id alignedlayer
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${ALIGNEDLAYER_PORT}657\"|" $HOME/.alignedlayer/config/client.toml
```
# 6. Download genesis and addrbook:
```bash
wget -O $HOME/.alignedlayer/config/genesis.json https://testnet-files.itrocket.net/alignedlayer/genesis.json
wget -O $HOME/.alignedlayer/config/addrbook.json https://testnet-files.itrocket.net/alignedlayer/addrbook.json
```
# 7. Set seeds and peers:
```bash
SEEDS="d1a8816c1c5800b352c2a1eb0e7a156bce34ae9f@alignedlayer-testnet-seed.itrocket.net:50656"
PEERS="596314959836975abce6dc56194e15efa51c49ce@62.169.17.52:26656,24235b3ac8bdc899b73ce46efb2478002c9578cd@49.13.210.129:26656,64140723ae47a79341ffccb302b2a34f4e0997c7@185.233.107.64:26656,81f355673d1ef4297682ffa991f537f6ef78fb7d@5.78.83.122:26656,dc2011a64fc5f888a3e575f84ecb680194307b56@148.251.235.130:20656,a1a98d9caf27c3363fab07a8e57ee0927d8c7eec@128.140.3.188:26656,1beca410dba8907a61552554b242b4200788201c@91.107.239.79:26656,f9000461b5f535f0c13a543898cc7ac1cd10f945@88.99.174.203:26656, 32fbefec592ac2ff9ecb3cad69bafaaad01e771a@148.251.235.130:20656,81138177a67195791bbe782fe1ed49f25e582bac@91.107.239.79:26656,c5d0498e345725365c1016795eecff4a67e4c4c9@88.99.174.203:26656,14af04afc663427604e8dd53f4023f7963a255cb@116.203.81.174:26656,9c89e77d51561c8b23957eee85a81ccc99fa7d6b@128.140.3.188:26656,c355b86c882d05a83f84afba379291d7b954b28f@65.108.236.43:26656,b499b9eb88c1c78ae25fdc7c390090f7542160eb@167.235.12.38:26656,18e1adeadb8cc596375e4212288fcd00690df067@213.199.48.195:26656,6d7adb46e588bea496f33758e0448bf84e308b39@143.244.178.205:26656,de193ba0ae387fc7892c2ead7458202f1c035d69@38.242.137.235:26656,5aaa5b73b2c39a7f60ff8bd952f2128d87af0e41@65.21.156.146:26656,48137de08aa2473eb6a5c9895037af055d4bdc73@138.197.81.13:26656,57028ecfafea4d760d5ed0a8a1fa648e817758b0@188.34.154.168:26656,c6ff629f95e12a6c142b73cafd7de78b8c39abaa@167.235.193.218:26656,b118ea4b07b17b4902f3aa3c52c209077cd96f1f@65.108.152.102:26656,d8a2b6dbf54bfb8fbbfc0365031968a55abbcd39@213.136.84.124:26656,6c096a3c475ac60bed346000f504882582eb8936@64.226.121.35:26656,ebd1f22fd96abacd1a8416c0e503c5a004257417@164.92.84.117:26656,c7dc853f74c39e24826a850f1c5a3108d605a71c@23.88.50.27:26656,cddc72c6ca172a6abe2af0b1a43bad16e3ea9b3f@65.109.143.157:26656,4fe77a8dbdb3451c45bea6f54dd5da0a0527829d@49.13.73.144:26656,d8a2b6dbf54bfb8fbbfc0365031968a55abbcd39@213.136.84.124:26656,3f76afabc393ead6b2f1e4f050016a546279d605@62.169.20.128:26656,8b0a89ff1add0c92a1e83b8790e6262cdebefbf5@37.27.86.49:26656,b118ea4b07b17b4902f3aa3c52c209077cd96f1f@65.108.152.102:26656,4e48dc6a9b8b1d33cbaac87fbba909b06f28a46a@207.180.210.121:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.alignedlayer/config/config.toml
```
# 8. Set custom ports in app.toml:
```bash
sed -i.bak -e "s%:1317%:${ALIGNEDLAYER_PORT}317%g;
s%:8080%:${ALIGNEDLAYER_PORT}080%g;
s%:9090%:${ALIGNEDLAYER_PORT}090%g;
s%:9091%:${ALIGNEDLAYER_PORT}091%g;
s%:8545%:${ALIGNEDLAYER_PORT}545%g;
s%:8546%:${ALIGNEDLAYER_PORT}546%g;
s%:6065%:${ALIGNEDLAYER_PORT}065%g" $HOME/.alignedlayer/config/app.toml
```
# 9. Set custom ports in config.toml file
```bash
sed -i.bak -e "s%:26658%:${ALIGNEDLAYER_PORT}658%g;
s%:26657%:${ALIGNEDLAYER_PORT}657%g;
s%:6060%:${ALIGNEDLAYER_PORT}060%g;
s%:26656%:${ALIGNEDLAYER_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ALIGNEDLAYER_PORT}656\"%;
s%:26660%:${ALIGNEDLAYER_PORT}660%g" $HOME/.alignedlayer/config/config.toml
```
# 10. Config pruning
```bash
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.alignedlayer/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.alignedlayer/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.alignedlayer/config/app.toml
```
# 11. Set minimum gas price, enable prometheus and disable indexing:
```bash
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.0001stake"|g' $HOME/.alignedlayer/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.alignedlayer/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.alignedlayer/config/config.toml
```
# 12. Create service file
```bash
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
```

# 13. Reset and download snapshot:
```bash
alignedlayerd tendermint unsafe-reset-all --home $HOME/.alignedlayer
if curl -s --head curl https://testnet-files.itrocket.net/alignedlayer/snap_alignedlayer.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://testnet-files.itrocket.net/alignedlayer/snap_alignedlayer.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.alignedlayer
    else
  echo no have snap
fi
```

# 14. Enable and start service: 
```bash
sudo systemctl daemon-reload
sudo systemctl enable alignedlayerd
sudo systemctl restart alignedlayerd && sudo journalctl -u alignedlayerd -f
```

# 15. Create wallet:

## To create a new wallet, use the following command. don’t forget to save the mnemonic
```bash
alignedlayerd keys add $WALLET
```
## To restore exexuting wallet, use the following command
```bash
alignedlayerd keys add $WALLET --recover
```
## Save wallet and validator address
```bash
WALLET_ADDRESS=$(alignedlayerd keys show $WALLET -a)
VALOPER_ADDRESS=$(alignedlayerd keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Check sync status, once your node is fully synced, the output from above will print "false"
```bash
alignedlayerd status 2>&1 | jq 
```
## Before creating a validator, you need to fund your wallet and check balance
```bash
alignedlayerd query bank balances $WALLET_ADDRESS 
```
# 16. Create validator: 
*Đổi tên theo tùy chọn: 
- moniker: tên node
- identity: public key của KEYBASE (nếu có), 
- website: nếu có
- security-contract: thông tin liên hệ
- details: miêu tả

```bash
cd $HOME
## Create validator.json file
echo "{\"pubkey\":{\"@type\":\"/cosmos.crypto.ed25519.PubKey\",\"key\":\"$(alignedlayerd comet show-validator | grep -Po '\"key\":\s*\"\K[^"]*')\"},
    \"amount\": \"900000stake\",
    \"moniker\": \"JosephTran\",
    \"identity\": \"9C3D03B4E6CA60CC\",
    \"website\": \"www.josephtran.xyz\",
    \"security\": \"@0josephtran102\",
    \"details\": \"I love blockchain ❤️\",
    \"commission-rate\": \"0.05\",
    \"commission-max-rate\": \"0.2\",
    \"commission-max-change-rate\": \"0.01\",
    \"min-self-delegation\": \"1\"
}" > validator.json
```
## Create a validator using the JSON configuration:
```bash
alignedlayerd tx staking create-validator validator.json \
    --from $WALLET \
    --chain-id alignedlayer \
	--fees 50stake \

    alignedvaloper10zgz5lnt3g44f65g0xrez82jyhaye75gjkg9e7
```
