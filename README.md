![image](https://github.com/molla202/Entangle/assets/91562185/e3546025-de57-498c-b68f-0d319a3d429f)



https://discord.gg/entanglefi
## Gereklilikleri kuralım
```
apt update && apt upgrade -y

apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```
## Go Kurulumu Yapıyoruz
```
cd $HOME
VER="1.19.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm -rf  "go$VER.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
go version
```

### Dosyaları çekiyoruz
```
rm -rf entangle-blockchain
git clone https://github.com/Entangle-Protocol/entangle-blockchain.git
cd entangle-blockchain
make install
```

### İnit işlemi : moniker yazınz
```
entangled config chain-id entangle_33033-1
entangled config keyring-backend file
entangled init "$NODE_MONIKER" --chain-id entangle_33033-1
```
```
curl -s https://raw.githubusercontent.com/Entangle-Protocol/entangle-blockchain/main/config/genesis.json > $HOME/.entangled/config/genesis.json
```
### ayarlamalar
```
SEEDS=""
PEERS="7ca810f11ddd54e7a10fed03a067b0ec3c1e8ba2@34.174.1.1:26656,2831d7b924b2b2774f70dbda6ebc8d8aa6f39e5d@95.216.75.190:26656,fbe38cb0500221859e631a93d7f830babea8871c@94.76.242.57:26656,1731c1b2f3bc8316a74d645a3f045c0f1aba7bc5@94.76.242.68:26656,841380487cf89030e9ae865dfb884cc703e6c2b2@94.76.242.98:26656,f381ea4431ba812caf27f65f1fbd31b64a699358@38.146.3.231:27156,6662a6b320b8f7520ebbc04e2891ee43227631a8@65.21.65.254:1550,9d25b1205658df64b509c21bcb212159f13039e3@37.252.186.204:26656,990d17a2686b47fe4bcf885bfae3f29897a58a32@65.108.71.227:26656,01f693f3f219e4fc58f4161c377a63011d369243@37.27.109.36:26656,da8baf26d486b9dd268dc62dd962141592f40068@65.108.97.119:26656,d47a5d94defcfa5eaac0dfc136b227a215a8f2b9@37.27.109.45:26656,8a6be04c58c3e633f9d3d201fd03d23b6cfcb3d1@65.21.33.59:26656,75e62df2ed4615cabe38a7b2d645e329a800be47@185.205.246.212:26656"
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.entangled/config/config.toml

sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.entangled/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.entangled/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.entangled/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 0|g' $HOME/.entangled/config/app.toml

sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.025aNGL"|g' $HOME/.entangled/config/app.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.entangled/config/config.toml
```
### Servis oluşturuyoruz
```
sudo tee /etc/systemd/system/entangled.service > /dev/null << EOF
[Unit]
Description=Entangle Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which entangled) start
Restart=on-failure
RestartSec=10
LimitNOFILE=10000
[Install]
WantedBy=multi-user.target
EOF
```
## snap

### Port değiştirmek isterseniz (opsiyonel)
```
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:14658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:14657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:14660\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:14656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":14666\"%" $HOME/.entangled/config/config.toml

sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:14617\"%; s%^address = \":8080\"%address = \":14680\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:14690\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:14691\"%; s%:8545%:14645%; s%:8546%:14646%; s%:6065%:14665%" $HOME/.entangled/config/app.toml
```
### Başlatıyoruz
```
sudo systemctl daemon-reload
sudo systemctl enable entangled
sudo systemctl restart entangled && sudo journalctl -u entangled -fo cat
```
### Private key alıp MM eklıyoruz...
NOT: MM ekledikten sonra çıkan adresle Discorddan faucet kanalına verify diyoruz bot mesaj atıyor ona adresi yazıyoruz oda yolluyor
```
entangled keys unsafe-export-eth-key cüzdan-adı
```

### cüzdan
```
entangled keys add $KEY --keyring-backend file --algo eth_secp256k1
```
### Validator olusturuyoruz
```
entangled tx staking create-validator \
--amount="5000000000000000000aNGL" \
--pubkey=$(entangled tendermint show-validator) \
--moniker="validator" \
--chain-id=entangle_33033-1 \
--commission-rate="0.10" \
--commission-max-rate="0.20" \
--commission-max-change-rate="0.01" \
--min-self-delegation="1" \
--gas=500000 \
--gas-prices="10aNGL" \
--from=<key_name>
```
### Delege
```
entangled tx staking delegate <TO_VALOPER_ADDRESS> 5000000000000000000aNGL --from $WALLET --chain-id entangle_33033-1 --gas=500000 --gas-prices="10aNGL" -y
```
### Redelege
```
entangled tx staking redelegate $VALOPER_ADDRESS <TO_VALOPER_ADDRESS> 5000000000000000000aNGL --from $WALLET --chain-id entangle_33033-1 --gas=500000 --gas-prices="10aNGL" -y
```
### unjail
```
entangled tx slashing unjail --from molla202 --chain-id entangle_33033-1 --gas-prices="10aNGL" -y
```
