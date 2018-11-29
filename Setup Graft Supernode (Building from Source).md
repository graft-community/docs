# Setup Graft Supernode (Building from Source)

## Getting Started
These instructions will help you build and setup a Graft Supernode from Source.

## Requirements
* OS:  Ubuntu 18.04 (other linux distributions and versions may require additional steps)
* RAM:  2GB per CPU Core
* If you are running from a local network you will also need to forward ports 28680 & 28681 to your supernode's local IP address.

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
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
````

## Add Firewall


````bash
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw allow 28680/tcp            # p2p node port
sudo ufw allow 28681/tcp            # rpc node port
sudo ufw allow 28690/tcp            # rpc supernode port
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
````

## Build Graft Supernode from source

````bash
cd ~
mkdir -p supernode/logs
cd supernode
cmake -DENABLE_SYSLOG=ON ~/graft-ng
make -j2    # Adjust as needed: -j1 for a low-powered machine, -j8 for an 8-core monster with at least 16GB ram
````

## Download current blockchain (optional to reduce time to sync blockchain)


````bash
mkdir -p ~/.graft/testnet/lmdb
wget https://rta.graft.observer/lmdb/data.mdb -P ~/.graft/testnet/lmdb/
````

## Start graftnoded

````bash
~/supernode/BUILD/bin/./graftnoded --testnet --log-file ~/supernode/logs/graftnoded.log --log-level=1 --detach
````

## Start graft_server
Start graft_server for long enough to create the stake-wallet then close it.  When it starts to scroll hit CTRL-C to exit.

````bash
~/supernode/./graft_server
````

## Add Stake to Stake-Wallet
The following command will give you the stake wallet address.  Use this address to request testnet graft in the Telegram group or Discord channel.

````bash
cat ~/.graft/supernode/data/stake-wallet/stake-wallet.address.txt
````

## Download Watch-Only -Wallets (Optional to help catch up graft_server)

````bash
mkdir -p ~/.graft/supernode/data
wget https://rta.graft.observer/lmdb/watch-only-wallets.tar
tar xf watch-only-wallets.tar -C ~/.graft/supernode/data
rm watch-only-wallets.tar
````

## Start Graft Server

````bash
~/supernode/./graft_server --log-level=1 --log-file ~/supernode/logs/graft_server.log
````

## Add graftnoded and graftserver services

Mae sure that graft_server and graftnoded are not running
````bash
~/supernode/BUILD/bin/./graftnoded --testnet exit
killall -9 graftnoded
killall -9 graft_server
````
  
Create graftnoded service using nano.
````bash
sudo nano /etc/systemd/system/graftnoded.service
````

Copy the following into the file that you just graftnoded.service.  Make sure to change graft-user to your username in all 4 places (User, WorkingDirectory, ExecStart X 2).  Use CTRL-X to save and exit.
````bash
[Unit]
Description=Graft node (RTA alpha)
After=network-online.target

[Service]
User=graft-user
Type=simple
WorkingDirectory=/home/graft-user/
Restart=always
ExecStart=/home/graft-user/supernode/BUILD/bin/./graftnoded --testnet --log-file /home/graft-user/supernode/logs/graftnoded.log --log-level=1 --non-interactive
LimitNOFILE=8192
Environment=TERM=xterm

[Install]
WantedBy=multi-user.target
````
  
Create graftserver service using nano.
````bash
sudo nano /etc/systemd/system/graftserver.service
````
Copy the following into the file that you just graftserver.service.  Make sure to change graft-user to your username in all 4 places (User, WorkingDirectory, ExecStart X 2).  Use CTRL-X to save and exit.
````bash
[Unit]
Description=Graft supernode (RTA alpha)
After=graftnoded.service

[Service]
User=graft-user
Type=simple
WorkingDirectory=/home/graft-user/supernode/
Restart=always
ExecStart=/home/graft-user/supernode/./graft_server  --log-level=1 --log-file /home/graft-user/supernode/logs/graft_server.log
Environment=TERM=xterm
LimitNOFILE=8192

[Install]
WantedBy=multi-user.target
````
Start and enable these services to start on boot.

````bash
sudo systemctl start graftnoded
sudo systemctl start graftserver
sudo systemctl enable graftnoded
sudo systemctl enable graftserver
sudo reboot
````

After reboot make sure services are running properly.

````bash
sudo systemctl status graftnoded
sudo systemctl status graftserver
````

