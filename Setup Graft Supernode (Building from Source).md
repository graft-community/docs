# Setup Graft Supernode (Building from Source)

## Getting Started
These instructions will help you build and setup a Graft Supernode from Source.

## Requirements
* OS:  Ubuntu 18.04 (other linux distributions and versions may require additional steps)
* RAM:  2GB per CPU Core

## Prepare the server
If you have not done so add a user and give it sudo rights.  We'll use graft-user, but you can use any username you prefer.

````bash
adduser graft-user
usermod -aG sudo graft-user
````

Logout of root and log back in with your new username and password.  Once you are logged in under the new account update and upgrade the server.  If you receive a prompt "What do you want to do about modified configuration file sshd_config?" choose the first option "install the package maintainer's version"

````bash
sudo apt-get update
sudo apt-get upgrade -y
````

## Create a swap file

````bash
free -h
df -h
sudo fallocate -l 4G /swapfile          # Change 4G to the proper size swap file for your server
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo su -
cat <<EOF >> /etc/fstab
/swapfile none swap sw 0 0
EOF
exit
````

## Add Firewall

````bash
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw allow 28600/tcp            # Load balancer http port
sudo ufw allow 28643/tcp            # Load balancer https port
sudo ufw allow 28680/tcp            # p2p node port
sudo ufw allow 28680/udp            # p2p node port
sudo ufw allow 28681/tcp            # rpc node port
sudo ufw allow 28690/tcp            # rpc supernode port
sudo ufw allow 8081/tcp             # block explorer port
sudo ufw logging on
sudo ufw -f enable
sudo ufw status
````

## Install prerequisites

````bash
sudo apt-get install -y git build-essential cmake pkg-config libboost-all-dev libssl-dev \
autoconf automake check libpcre3-dev rapidjson-dev libreadline-dev
````

## Increase the limit for maximum allowed opened file descriptors per process

````bash
sudo sed -i -e 's/.*DefaultLimitNOFILE=.*/DefaultLimitNOFILE=8192/g' /etc/systemd/system.conf
````


## Git Graft

````bash
git clone --recursive -b alpha3 https://github.com/graft-project/graft-ng.git
cd graft-ng
git submodule update --init --recursive
````

## Build Graft Supernode from source

````bash
cd $HOME
mkdir -p supernode/logs
cd supernode
cmake -DENABLE_SYSLOG=ON $HOME/graft-ng
make -j2    # Adjust as needed: -j1 for a low-powered machine, -j8 for an 8-core monster with at least 16GB ram
````

## Download current blockchain (optional to reduce time to sync blockchain)


````bash
mkdir -p $HOME/.graft/testnet/lmdb
wget https://rta.graft.observer/lmdb/data.mdb -P $HOME/.graft/testnet/lmdb/
````

## Start graftnoded

````bash
$HOME/supernode/BUILD/bin/./graftnoded --testnet --log-file ~/supernode/logs/graftnoded.log --log-level=1 --detach
````

## Start graft_server
Start graft_server for long enough to create the stake-wallet then close it.  When it starts to scroll hit CTRL-C to exit.

````bash
$HOME/supernode/./graft_server
````

## Add Stake to Stake-Wallet
The following command will give you the stake wallet address.  Use this address to request testnet graft in the Telegram group or Discord channel.

````bash
cat $HOME/.graft/supernode/data/stake-wallet/stake-wallet.address.txt
````

## Download Watch-Only -Wallets (Optional to help catch up graft_server)

````bash
mkdir -p $HOME/.graft/supernode/data
wget https://rta.graft.observer/lmdb/watch-only-wallets.tar
tar xf watch-only-wallets.tar -C $HOME/.graft/supernode/data
rm watch-only-wallets.tar
````

## Start Graft Server

````bash
$HOME/supernode/./graft_server --log-level=1 --log-file $HOME/supernode/logs/graft_server.log
````
