
# Verify for Ubuntu 22.04

```sh
apt update && apt install build-essential -y

apt update && apt dist-upgrade -y

apt install -y mc htop jq curl screen git unzip python3-pip tmux libssl-dev libffi-dev python3-dev libsnappy-dev libleveldb-dev make bash-completion ufw
```

## Go Installation (see the latest version of Go on https://go.dev/dl/)
```sh
wget https://go.dev/dl/go1.22.3.linux-amd64.tar.gz && \
rm -rf /usr/local/go && \
tar -C /usr/local -xzf go1.22.3.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> /etc/profile
source /etc/profile
```

## Check that the version is correct
```sh
go version
```

## dscd Installation (see the latest version on https://repo.decimalchain.com/)
### The current preferred option is to download a binary for your Linux distribution.
```sh
wget https://repo.decimalchain.com/15656601/linux/ubuntu/22.04/dscd && \
sudo mkdir -p $HOME/go/bin && sudo mv dscd $HOME/go/bin && \
sudo chmod +wx $HOME/go/bin/dscd
```

## Check that the version is correct
```sh
dscd version
```

## If the node has already been in use, it is necessary to make a backup copy of the existing configuration, usually in the folder $HOME/.decimal/daemon/config
### After backup:
```sh
rm -rf "$HOME/.decimal/daemon"
```

## Choose a name for your node
```sh
NODE_MONIKER="node"
```

## Get the current chain_id from Blockchain
```sh
CHAIN_ID="$(curl -s 'https://mainnet-gate.decimalchain.com/api/rpc/genesis/chain')"
```

## Initialize your node
```sh
dscd init "$NODE_MONIKER" --network mainnet --chain-id "$CHAIN_ID"
```

## Get the genesis
```sh
wget -q https://repo.decimalchain.com/genesis.tar.xz && \
tar -xvf genesis.tar.xz && \
mv genesis/genesis.json $HOME/.decimal/daemon/config && \
rm -r ./genesis/ && \
rm -rf genesis.tar.xz
```

## Get the daily backup of the blockchain
```sh
rsync -avPh rsync://rsync-mainnet.decimalchain.com:10000/daily-data/ $HOME/.decimal/daemon/data/ --delete
```

## Configure /$HOME/.decimal/daemon/config/config.toml
### Change seeds parameter
```toml
seeds = "0906b583daebe8951226e56cf75e1d2175f19671@decimal-node-1.mainnet.decimalchain.com:26656,1e9a5adb32f39a62849c94dbec95f251f5ebd728@decimal-node-2.mainnet.decimalchain.com:26656"
```

## Launch the node after synchronization
```sh
dscd start --trace
```

## Add a service (don't forget the correct path for dscd service)
```sh
bash -c 'cat <<EOM >/lib/systemd/system/decimal.service
[Unit]
Description=Decimal Node daemon
Documentation=https://help.decimalchain.com
Requires=network-online.target
After=network-online.target

[Service]
User=root
Group=root
WorkingDirectory=/root/.decimal/daemon/
Type=simple
Restart=always
RestartSec=3
ExecStart=/root/go/bin/dscd start --trace
StandardOutput=journal
StandardError=journal
SyslogIdentifier=DSCD
ExecReload=/bin/kill -HUP 
ExecStop=/bin/kill 
KillSignal=SIGTERM
LimitNOFILE=1000000
LimitNPROC=1000000

[Install]
WantedBy=multi-user.target
EOM'

systemctl daemon-reload
systemctl enable decimal
```

## Ulimits
```sh
bash -c 'cat <<EOM >>/etc/security/limits.conf
*         hard    nofile      1000000
*         soft    nofile      1000000
root      hard    nofile      1000000
root      soft    nofile      1000000
EOM'
```
```sh
bash -c 'cat <<EOM >>/etc/pam.d/common-session
session required pam_limits.so
EOM'
```
```sh
bash -c 'cat <<EOM >>/etc/sysctl.conf
fs.file-max = 2097152
EOM'
```

## Firewall
```sh
ufw default deny incoming
ufw default allow outgoing
ufw limit 22/tcp comment "SSH"
ufw allow 26656/tcp comment "decimal node p2p"
echo "y" | ufw enable
```

## Reboot and check connections
```sh
reboot
```

## Log filters for new blocks and guard messages
```sh
journalctl -a -f | grep -E "indexed block exents|GUARD"
```

# Launch Guard
```sh
cd $HOME
git clone https://bitbucket.org/decimalteam/dsc-guard.git

cd $HOME/dsc-guard/cmd/gentx/
cp .env_example .env
```

## Edit the env file (put the mnemonic of your validator wallet)
```env
DECIMAL_GATEWAY="https://mainnet-gate.decimalchain.com/api"
MNEMONIC="YOUR MNEMONIC"
```

## Run Guard, the first time it will download all needed packages
```sh
go run main.go  # copy transaction hash and put it into the guard .env file
```

## Configure Guard (the first time)
```sh
cd $HOME/dsc-guard/cmd/dsc-guard/
cp .env_example .env
```

## Configure the env file

```env
NODES_ENDPOINTS=tcp://localhost:26657,tcp://abcd:26657 
MISSED_BLOCKS_LIMIT=8
MISSED_BLOCKS_WINDOW=24
FALLBACK_PAUSE=2
NEW_BLOCK_TIMEOUT=10
VALIDATOR_ADDRESS=
HTTP_LISTENER=localhost:11111
SET_OFFLINE_TX=0a5b0a590a232...............6200
```

- `NODES_ENDPOINTS` - comma-separated list of nodes in the DSC blockchain with open Tendermint ports (26657)
- `MISSED_BLOCKS_LIMIT` and `MISSED_BLOCKS_WINDOW` - upon reaching how many missed blocks `MISSED_BLOCKS_LIMIT` in a window of size `MISSED_BLOCKS_WINDOW`, a set-offline transaction should be sent
- `FALLBACK_PAUSE` - time to reconnect to nodes in seconds
- `NEW_BLOCK_TIMEOUT` - time to wait for a new block in seconds
- `VALIDATOR_ADDRESS` - Tendermint address of the validator, taken from `priv_validator_key.json`, field "address"
- `SET_OFFLINE_TX` - hex representation of the set-offline transaction, should be the result of running `gentx`
- `HTTP_LISTENER` - optional. If set as host:port, then JSON is issued at this address according to the status of the guard

## Try to run Guard
```sh
cd $HOME/dsc-guard/cmd/dsc-guard/
go run main.go
```

You will see something like this:
```
I[2024-05-29|16:46:20.683] [http://localhost:26657] Retrieved set of validators for block 20864248 
I[2024-05-29|16:46:20.683] [http://localhost:26657] validator not in set: 351475A89CD********9D8A3C8D4B3AF  # it's about the validator not set in blockchain yet, and it's okay now
I[2024-05-29|16:46:20.684] [http://localhost:26657] Check set_offline transaction ok 
I[2024-05-29|16:46:25.781] [http://localhost:26657] Retrieved signatures for block 20864248 
```

## Install Guard as a service (don't forget to change the path to Go and dsc-guard)
```sh
bash -c 'cat <<EOM >/lib/systemd/system/guard.service
[Unit]
Description=Decimal Guard daemon
Documentation=https://help.decimalchain.com

[Service]
Type=idle
User=root
ExecStart=/usr/local/go/bin/go run /root/dsc-guard/cmd/dsc-guard/main.go
Restart=always
RestartSec=3s
StandardOutput=journal
StandardError=journal
SyslogIdentifier=GUARD
Environment=GOROOT=/usr/local/go
Environment=GOPATH=/root/go
WorkingDirectory=/root/dsc-guard/cmd/dsc-guard
[Install]
WantedBy=multi-user.target
EOM'

systemctl daemon-reload
systemctl enable guard
```

## Restart the system and check that everything is fine
```sh
reboot
```

## After the system is up, run dscd up for about 2-10 minutes, and it will need to sync
```sh
journalctl -a -f | grep -E "indexed block exents|GUARD"
```

## You will see

 something like this
```
May 29 16:56:20 ValidatorName DSCD[314]: 4:56PM INF indexed block events height=20864355 module=txindex server=node
May 29 16:56:25 ValidatorName GUARD[618]: I[2024-05-29|16:56:25.387] [http://localhost:26657] Retrieved signatures for block 20864355
May 29 16:56:25 ValidatorName GUARD[618]: D[2024-05-29|16:56:25.387] missed blocks in window = 0
May 29 16:56:25 ValidatorName GUARD[618]: I[2024-05-29|16:56:25.389] [http://localhost:26657] Retrieved set of validators for block 20864356
May 29 16:56:25 ValidatorName GUARD[618]: I[2024-05-29|16:56:25.389] [http://localhost:26657] validator not in set: 351475A8**********9D8A3C8D4B3AF
May 29 16:56:25 ValidatorName GUARD[618]: I[2024-05-29|16:56:25.390] [http://localhost:26657] Check set_offline transaction ok
```

## If you see this, the system is ready to use.
```
