# Althea Mainnet full node setup guide

1. Update the system:
```
sudo apt update && sudo apt upgrade --yes
```
2. Install core tools:
```
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils htop net-tools lsof --yes
```
3. Install Go:
```
cd $HOME
wget -c -O go1.23.4.linux-amd64.tar.gz https://golang.org/dl/go1.23.4.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz && sudo rm go1.23.4.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
sudo cp $(which go) /usr/local/bin
go version
```
4. Clone the Althea software:
```
cd $HOME
git clone https://github.com/AltheaFoundation/althea-L1.git
cd althea-L1
git pull
```
5. Checkout to the actual tag:
```
git checkout v6.7.0
```
6. Build the software and copy ``althea`` binary file to ``/usr/local/bin``:
```
make build
sudo cp $HOME/althea-L1/build/althea /usr/local/bin
althea version
```
7. Initialize ``althea``:
```
althea init <your_moniker> --chain-id althea_417834-4
```
8. Get the genesis file:
```
wget -O $HOME/.althea/config/genesis.json https://github.com/AltheaFoundation/althea-L1-docs/raw/main/althea-l1-mainnet-genesis.json
```
9. Add peers to ``$HOME/.althea/config/config.toml``:
```
peers="4d9c73a9e541453b56add8fadf0839fd1442d979@15.235.115.155:17200,a0eca501485cc74e0568973ef502d05023f6500d@158.247.226.255:17200,ab9a9e6ea747839652dfe4480e66a5eb78a385e8@51.81.167.60:17200"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.althea/config/config.toml
```
10. Configure the background service:
```
sudo tee /etc/systemd/system/althea.service > /dev/null <<'EOF'
[Unit]
Description=Althea daemon
After=network-online.target

[Service]
User=root
ExecStart=($which althea) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
11.  Enable service and start your node:
```
sudo systemctl daemon-reload
sudo systemctl enable althea
sudo systemctl restart althea
journalctl -u althea -f -o cat
```
12. If you want to sync your full node quickly you can use state sync guide here: [Althea State Sync](https://github.com/mediumwe11/Althea-Mainnet-Guides/blob/main/statesync.md)
