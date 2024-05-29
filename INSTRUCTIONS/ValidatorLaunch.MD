## Verify for Ubuntu 22.04


apt update && apt install build-essential -y

apt update && apt dist-upgrade -y

apt install -y mc htop jq curl screen git unzip python3-pip tmux libssl-dev libffi-dev python3-dev libsnappy-dev libleveldb-dev make bash-completion ufw

## go install (see latest version of go on https://go.dev/dl/ )
wget https://go.dev/dl/go1.22.3.linux-amd64.tar.gz && \
rm -rf /usr/local/go && \
tar -C /usr/local -xzf go1.22.3.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:/$HOME/go/bin' >> /etc/profile
source /etc/profile

##check that version is correct
go version

## dscd install (see latest version of go on https://repo.decimalchain.com/)
## The current preferred option is to download a binary for your linux distribution.
wget https://repo.decimalchain.com/15656601/linux/ubuntu/22.04/dscd && \
sudo mkdir -p /$HOME/go/bin && sudo mv dscd /$HOME/go/bin && \
sudo chmod +wx /$HOME/go/bin/dscd

##check that version is correct
dscd version

##if node alreade been in use, it is necessary to make a backup copy of the existing configuration, usually it is the folder $HOME/.decimal/daemon/config
##after backup:
rm -rf "$HOME/.decimal/daemon"
# Chose a name of yor node
NODE_MONIKER="node" 
# Get cuurent chain_id from Blockchain
CHAIN_ID="$(curl -s 'https://mainnet-gate.decimalchain.com/api/rpc/genesis/chain')"
# Initializate your node
dscd init "$NODE_MONIKER" --network mainnet --chain-id "$CHAIN_ID"
# And get a genesis
wget -q  https://repo.decimalchain.com/genesis.tar.xz && \
tar -xvf genesis.tar.xz && \
mv genesis/genesis.json $HOME/.decimal/daemon/config && \
rm -r ./genesis/ && \
rm -rf genesis.tar.xz


## Get daily backup of blockchain
rsync -avPh rsync://rsync-mainnet.decimalchain.com:10000/daily-data/ $HOME/.decimal/daemon/data/ --delete


## config /$HOME/.decimal/daemon/config/config.toml 
seeds = "0906b583daebe8951226e56cf75e1d2175f19671@decimal-node-1.mainnet.decimalchain.com:26656,1e9a5adb32f39a62849c94dbec95f251f5ebd728@decimal-node-2.mainnet.decimalchain.com:26656"

## launch node after sync
dscd start --trace

## add service (dont forget for right path for dscd service)
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

## ulimits
bash -c 'cat <<EOM >>/etc/security/limits.conf
*         hard    nofile      1000000
*         soft    nofile      1000000
root      hard    nofile      1000000
root      soft    nofile      1000000
EOM'

bash -c 'cat <<EOM >>/etc/pam.d/common-session
session required pam_limits.so
EOM'

bash -c 'cat <<EOM >>/etc/sysctl.conf
fs.file-max = 2097152
EOM'

## firewall
ufw default deny incoming
ufw default allow outgoing
ufw limit 22/tcp comment "SSH"
ufw allow 26656/tcp comment "decimal node p2p"
echo "y" | ufw enable

## reboot and chec connections
reboot

## log filters for new blocks and guard messages
journalctl -a -f | grep -E "indexed block exents|GUARD"


## Launch Guard

git clone https://bitbucket.org/decimalteam/dsc-guard.git
