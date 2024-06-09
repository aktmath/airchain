# airchain
Min Req : 
CPU 	2   |
RAM 	4+ GB   |
Storage 	100+ GB SSD   |

# necessary setups

````
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip screen -y
sudo apt install -y curl git jq lz4 build-essential cmake perl automake autoconf libtool wget libssl-dev -y
````

````
ufw allow 16545
````

# Go

````
cd $HOME && \
ver="1.21.3" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
````

````
git clone https://github.com/airchains-network/evm-station.git
````



````
wget http://37.120.189.81/airchain_testnet/tracks
chmod +x tracks
````

for ubuntu.20
````
wget http://37.120.189.81/airchain_testnet/ubuntu20/tracks
chmod +x tracks
````

# Evm-Station

````
cd evm-station
````

````
go mod tidy
````


````
nano /root/evm-station/scripts/local-setup.sh
````

Change the Chain ID, I made it like this : aktmath_1234-1

````
/bin/bash ./scripts/local-setup.sh
````
Save your wallet infos

# Port set-up

````
echo "export G_PORT="16"" >> $HOME/.bash_profile
source $HOME/.bash_profile
````

````
sed -i.bak -e "s%:1317%:${G_PORT}317%g;
s%:8080%:${G_PORT}080%g;
s%:9090%:${G_PORT}090%g;
s%:9091%:${G_PORT}091%g;
s%:8545%:${G_PORT}545%g;
s%:8546%:${G_PORT}546%g;
s%:6065%:${G_PORT}065%g" $HOME/.evmosd/config/app.toml
````

````
sed -i.bak -e "s%:26658%:${G_PORT}658%g;
s%:26657%:${G_PORT}657%g;
s%:6060%:${G_PORT}060%g;
s%:26656%:${G_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${G_PORT}656\"%;
s%:26660%:${G_PORT}660%g" $HOME/.evmosd/config/config.toml
````

SAVE YOUR PRIVATE KEY
````
cd
cd evm-station
/bin/bash ./scripts/local-keys.sh
````

# Service

````
CHAINID=YOUR CHAIN ID(mine: aktmath_1234-1)
````

````
sudo tee /etc/systemd/system/evmosd.service > /dev/null <<EOF
[Unit]
Description=evmosd node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.evmosd
ExecStart=/root/evm-station/build/station-evm start \
--metrics "" \
--log_level "info" \
--json-rpc.api eth,txpool,personal,net,debug,web3 \
--chain-id "$CHAINID"
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
````

````
sudo systemctl daemon-reload
sudo systemctl enable evmosd
sudo systemctl restart evmosd
````

# LOG

````
sudo journalctl -u evmosd -fo cat
````

# You will also need to set-up Avail Light Node

Avail Light Node : https://github.com/aktmath/avail-light-node

(Save your pubkey after setting Avail Light Node)

# Track

Add 0x in front of to your Avail pubkey

````
nano /root/.avail/identity/identity.toml
````

Go to this page : https://temp-explorer.avail.so/?rpc=wss%3A%2F%2Fturing-rpc.corenode.info#/accounts
Add account and enter  the keys which you got it from Avail node
Faucet : https://faucet.avail.tools

Moniker : Your moniker name

````
cd
````


````
./tracks init --daRpc "http://127.0.0.1:7000" --daKey "avail-pubkey-with-0x" --daType "avail" --moniker "enter-moniker-name" --stationRpc "http://127.0.0.1:16545" --stationAPI "http://127.0.0.1:16545" --stationType "evm"
````

Save all outro and ask faucet token from their discord : https://discord.gg/airchains

````
./tracks keys junction --accountName cüzdan-adi-yaz --accountPath $HOME/.tracks/junction-accounts/keys
````

# Prover

````
./tracks prover v1EVM
````

# Junction Station

Node-ID : Copy your node id from the file

````
nano /root/.tracks/config/sequencer.toml
````

wallet name : enter your wallet name
enter wallet address : your wallet address starts with air
/ip4/ENTER-VPS-IP-ADRESS/tcp/2300/p2p/ENTER-NODE-ID
Try again if you get any error

````
./tracks create-station --accountName WALLET-NAME --accountPath $HOME/.tracks/junction-accounts/keys --jsonRPC "https://airchains-testnet-rpc.cosmonautstakes.com/" --info "EVM Track" --tracks ENTER-WALLET-ADRESS --bootstrapNode "/ip4/ENTER-VPS-IP-ADRESS/tcp/2300/p2p/ENTER-NODE-ID"
````

# Start

````
screen -S etm
````


````
./tracks start
````
You will see this message : "RPC Server Stared at Port ... Succesfully module=rpc"
Then use Ctrl+A+D


DONT USE THIS PART IF ITS ALREADY WORKING

````
sudo tee /etc/systemd/system/tracksd.service > /dev/null <<EOF
[Unit]
Description=tracksd node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.tracks
ExecStart=/root/tracks start

Restart=always
RestartSec=10
LimitNOFILE=65535
SuccessExitStatus=0 1
[Install]
WantedBy=multi-user.target
EOF
````

````
sudo systemctl daemon-reload
sudo systemctl enable tracksd
sudo systemctl restart tracksd
````

````
sudo journalctl -u tracksd -fo cat
````

# Points

Enter your wallet keys to a Leap wallet
Check your points : https://points.airchains.io/

