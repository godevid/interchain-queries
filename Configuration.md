
<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/50621007/177221972-75fcf1b3-6e95-44dd-b43e-e32377685af8.png">
</p>

# Relay interchain-queries using the new GO v2 Relayer

## 1. Download and build binaries
```
ch $HOME
git clone https://github.com/Stride-Labs/interchain-queries.git
cd interchain-queries
go build
mv interchain-queries /usr/local/bin/icq
```

## 2. Make home dir for icq and create configurations file
```
cd $HOME && mkdir .icq
sudo tee $HOME/.icq/config.yaml > /dev/null <<EOF
default_chain: stride-testnet
chains:
  gaia-testnet:
    key: wallet
    chain-id: GAIA
    rpc-addr: http://<YOUR_GAIA_RPC_ADDR>:<YOUR_GAIA_RPC_PORT>         # Example: '127.0.0.1:23657'
    grpc-addr: http://<YOUR_GAIA_GRPC_ADDR>:<YOUR_GAIA_GRPC_PORT>      # Example: '127.0.0.1:23090'
    account-prefix: cosmos
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001uatom
    key-directory: /root/.icq/keys
    debug: false
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
  stride-testnet:
    key: wallet
    chain-id: STRIDE-TESTNET-2
    rpc-addr: http://<YOUR_STRIDE_RPC_ADDR>:<YOUR_STRIDE_RPC_PORT>      # Example: '127.0.0.1:16657'
    grpc-addr: http://<YOUR_STRIDE_RPC_ADDR>:<YOUR_STRIDE_GRPC_PORT>    # Example: '127.0.0.1:16090'
    account-prefix: stride
    keyring-backend: test
    gas-adjustment: 1.2
    gas-prices: 0.001ustrd
    key-directory: /root/.icq/keys
    debug: false
    timeout: 20s
    block-timeout: ""
    output-format: json
    sign-mode: direct
cl: {}
EOF
```

## 3. Import wallets
```
icq keys restore --chain stride-testnet wallet
icq keys restore --chain gaia-testnet wallet
```

## 4. Create icq service
```
sudo tee /etc/systemd/system/icqd.service > /dev/null <<EOF
[Unit]
Description=Interchain Query Service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which icq) run --debug
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## 5. Start icq service
```
sudo systemctl daemon-reload
sudo systemctl enable icqd
sudo systemctl restart icqd
```

## 6. Check icq logs
```
journalctl -u icqd -f -o cat
```

You will have to wait some time until you see some logs (5-15min):
```
store/bank/key
height parsed from GetHeightFromMetadata= 0
Fetching client update for height height 176886
store/bank/key
height parsed from GetHeightFromMetadata= 0
Fetching client update for height height 176886
Requerying lightblock
Requerying lightblock
Requerying lightblock
ICQ RELAYER | query.Height= 0
ICQ RELAYER | res.Height= 176885
Requerying lightblock
ICQ RELAYER | query.Height= 0
ICQ RELAYER | res.Height= 176885
Send batch of 4 messages
1 ClientUpdate message
1 SubmitResponse message
1 ClientUpdate message
1 SubmitResponse message
Sent batch of 2 (deduplicated) messages
```

After that you can check you transaction in the explorer

