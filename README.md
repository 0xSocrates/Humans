# Humans
Humans Testnet-1
### sistem gereksinimleri (resmi dökümanda yazanlar)
```
Memory: 8 GB RAM
CPU: Quad-Core
Disk: 250 GB SSD Storage
Bandwidth: 1 Gbps for Download / 100 Mbps for Upload
```
### sunucu güncellemesi
```
sudo apt update && sudo apt upgrade -y
```

```
sudo apt install curl tar wget tmux htop net-tools clang pkg-config libssl-dev jq build-essential git screen make ncdu -y
```
### go
```
cd $HOME
wget -O go1.18.4.linux-amd64.tar.gz https://golang.org/dl/go1.18.4.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.18.4.linux-amd64.tar.gz && rm go1.18.4.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
go version
```
### binary kurulumu
```
git clone https://github.com/humansdotai/humans
```
```
cd humans
```
```
git checkout v1.0.0
```
```
go build -o humansd cmd/humansd/main.go
```
```
mv humansd /usr/local/go/bin
```
### initalize
* monikerismi belirlemeyi unutmayın
```
humansd init monikerismi
```





### genesis ve addrbook
```
curl -s https://rpc-testnet.humans.zone/genesis | jq -r .result.genesis > genesis.json
```
```
mv genesis.json $HOME/.humans/config
```
```
wget -O $HOME/.humans/config/addrbook.json "https://raw.githubusercontent.com/sergiomateiko/addrbooks/main/humans/addrbook.json"
```
### yapılandırma dosyası
```
PEERS="1df6735ac39c8f07ae5db31923a0d38ec6d1372b@45.136.40.6:26656,9726b7ba17ee87006055a9b7a45293bfd7b7f0fc@45.136.40.16:26656,6e84cde074d4af8a9df59d125db3bf8d6722a787@45.136.40.18:26656,eda3e2255f3c88f97673d61d6f37b243de34e9d9@45.136.40.13:26656,4de8c8acccecc8e0bed4a218c2ef235ab68b5cf2@45.136.40.12:26656"
```
```
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.humans/config/config.toml
```
```
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.025uheart"/g' $HOME/.humans/config/app.toml
```
```
CONFIG_TOML="$HOME/.humans/config/config.toml"
sed -i 's/timeout_propose =.*/timeout_propose = "100ms"/g' $CONFIG_TOML
sed -i 's/timeout_propose_delta =.*/timeout_propose_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_prevote =.*/timeout_prevote = "100ms"/g' $CONFIG_TOML
sed -i 's/timeout_prevote_delta =.*/timeout_prevote_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_precommit =.*/timeout_precommit = "100ms"/g' $CONFIG_TOML
sed -i 's/timeout_precommit_delta =.*/timeout_precommit_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_commit =.*/timeout_commit = "1s"/g' $CONFIG_TOML
sed -i 's/skip_timeout_commit =.*/skip_timeout_commit = false/g' $CONFIG_TOML
```
### pruning ve indexer
* bu kısım opsiyonel. disk kullanımını düşürür aynı zamanda cpu ve ram kullanımını arttırır
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.humans/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.humans/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.humans/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.humans/config/app.toml
```
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.humans/config/config.toml
```







### systemd
```
sudo tee /etc/systemd/system/humansd.service > /dev/null <<EOF
[Unit]
Description=humans node
After=network.target
[Service]
User=root
Type=simple
ExecStart=$(which humansd) start --home $HOME/.humans
Restart=on-failure
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
```
systemctl daemon-reload
```
```
systemctl enable humansd
```
```
systemctl start humansd
```
### service durumu
```
systemctl status humansd
```
### logları görüntülemek
```
journalctl -u humansd -fo cat
```
### validatör oluşturmak için nodun ağ ile senkronize olmasını beklemek gerekiyor.
### güncel blok yüksekliğini takip etmek için: [explorer](https://explorer.humans.zone/humans-testnet)
### senkronizasyon durumu
* bu komutta `"catching_up": false` çıktısı senkronize olduğunu gösterir
```
humansd status 2>&1 | jq .SyncInfo
```
### cüzdan
* cüzdanismini belirlemeyi unutmayın
```
humansd keys add cüzdanismi
```
### [faucet](https://discord.gg/humansdotai)
### cüzdan bakiyesi kontrol
```
humansd q bank balances cüzdanadresi
```
### senkronizasyon tamamlandıysa artık validatör oluşturabilirsiniz
* `cüzdanismi`ve `monikerismi` değiştirmeyi unutmayın
```
humansd tx staking create-validator \
--amount 9000000uheart \
--from cüzdamismi \
--commission-max-change-rate "0.01" \
--commission-max-rate "0.2" \
--commission-rate "0.07" \
--min-self-delegation "1" \
--pubkey $(humansd tendermint show-validator) \
--moniker monikerismi \
--chain-id testnet-1 \
--fees 5050uheart
```








### node silmek için
```
sudo systemctl stop humansd
sudo systemctl disable humansd
rm -rf /etc/systemd/system/humansd.service
sudo rm -rf $(which humansd)
sudo rm -rf $HOME/.humansd
sudo rm -rf $HOME/humansd
```








yararlı komutlar: [link](https://forum.rues.info/index.php?threads/cosmos-aglarinda-kullanilan-ortak-komutlar.2540/) 

[website](https://humans.ai/)

[explorer](https://explorer.humans.zone/humans-testnet)

[türkçe telegram](https://t.me/+FgDavVRUXTgzNzc0)

[twitter](https://twitter.com/humansdotai)

[discord](https://discord.gg/humansdotai)














































