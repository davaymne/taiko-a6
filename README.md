# TAIKO TESTNET A6

This guide will help you start up a Taiko Node using simple-taiko-node and L1 Holeski Node.

## Run a Holesky node

### Run Geth execution layer

Reference: https://notes.ethereum.org/@launchpad/holesky

1. Prerequisite Go land and make

```
wget https://go.dev/dl/go1.21.6.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.6.linux-amd64.tar.gz
vi .bashrc << export PATH=$PATH:/usr/local/go/bin
sudo apt install make
```

2. Install Geth

```
git clone https://github.com/ethereum/go-ethereum.git
cd go-ethereum
make geth
```

3. Create jwt secret

```
openssl rand -hex 32 | tr -d "\n" > "/home/taiko/geth/jwtsecret"
```

4. Create service file

sudo vi /etc/systemd/system/geth.service
```
[Unit]
Description=ETH Holesky
After=network-online.target
[Service]
User=taiko
WorkingDirectory=/home/taiko/geth
[Unit]
Description=ETH Holesky
After=network-online.target
[Service]
User=taiko
WorkingDirectory=/home/taiko/geth
ExecStart=/home/taiko/geth/go-ethereum/build/bin/geth \
        --holesky \
        --datadir holesky \
        --authrpc.addr localhost \
        --authrpc.port 38551 \
        --authrpc.vhosts "*" \
        --authrpc.jwtsecret=/home/taiko/geth/jwtsecret \
        --http --http.addr 0.0.0.0 --http.api eth,net \
        --http.port 38545 \
        --metrics.expensive --metrics.addr 127.0.0.1 --metrics.port 6060 \
        --port 33303 --gcmode archive --syncmode full \
        --ws --ws.api debug,eth,net,web3,txpool --ws.addr "0.0.0.0" --ws.origins "*" \
        --ws.port 38545 \
        --rpc.txfeecap 10
StandardOutput=journal
StandardError=journal
Restart=always
RestartSec=3
StartLimitInterval=0
TimeoutStopSec=3600
LimitNOFILE=65536
LimitNPROC=65536
[Install]
WantedBy=multi-user.target
```

NOTE: check paths for files

5. Open ports on your firewall

```
8545 TCP, used by the HTTP based JSON RPC API
8546 TCP, used by the WebSocket based JSON RPC API
8547 TCP, used by the GraphQL API
33303 TCP and UDP, used by the P2P protocol running the network

```

6. Service commands

```
sudo systemctl enable --now geth.service
sudo systemctl restart geth.service
sudo journalctl -f -u geth.service
```

### Run Lighthouse consensus layer

1. Prerequisite

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"

sudo apt install build-essential cmake
```

2. Install Lighthouse

```
git clone https://github.com/sigp/lighthouse.git
cd lighthouse
make
```

3. Create service file
sudo vi  /etc/systemd/system/lighthouse.service 

```
[Unit]
Description=Ethereum CL (lighthouse)
After=network-online.target
[Service]
User=taiko
WorkingDirectory=/home/taiko/lighthouse
ExecStart=/home/taiko/.cargo/bin/lighthouse bn \
        --network holesky \
        --checkpoint-sync-url https://checkpoint-sync.holesky.ethpandaops.io \
        --datadir lighthouse-datadir \
        --execution-jwt /home/taiko/geth/jwtsecret \
        --execution-endpoint http://127.0.0.1:38551 \
        --metrics \
        --metrics-address 0.0.0.0 \
        --metrics-port 8080 \
        --port 29000 \
        --disable-upnp \
        --discovery-port 29001 \
        --disable-deposit-contract-sync \
        --disable-quic \
        --http
StandardOutput=journal
StandardError=journal
Restart=always
RestartSec=3
StartLimitInterval=0
TimeoutStopSec=3600
LimitNOFILE=65536
LimitNPROC=65536
[Install]
WantedBy=multi-user.target
```

4. Open ports on your firewall

```
29001
8080
5052
29000
```

5. Service commands

```
sudo systemctl enable --now lighthouse.service
sudo systemctl restart lighthouse.service
sudo journalctl -f -u lighthouse.service
```

## Run Taiko node

WARNING: Wait until L1 is fully synced

1. Clone git

```
git clone https://github.com/davaymne/taiko-a6.git
```

2. Copy the sample .env files into .env

```
cp .env.sample .env
```

3. Modify .env to set the L1 archive node endpoint

example:
```
L1_ENDPOINT_HTTP=http://<server IP>:38545
L1_ENDPOINT_WS=ws://<server IP>:38546
```

NOTE: Prover and Proposer is enabled by default

4. Run Taiko node

```
docker-compose up -d
```
