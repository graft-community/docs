# Setting up a working graft-wallet-cli

## An introduction to this guide:

Using the graft-cli-wallet is important because it is required at this time to stake your SuperNode and it is also the most secure form of wallet available, for more paranoid of users. 

Also consider that we are using the official build in this guide. There also may be newer versions of the wallet available when YOU are reading this, so best go check here [Graft Network Official Build](https://github.com/graft-project/GraftNetwork/releases)

*There are two ways you can run the graft-wallet-cli.* 

- 1) **Point it at a remote trusted machine** [**Creating a New Wallet and connecting to the remote node**](#creating-a-new-wallet-or-restoring-an-existing-wallet-and-connecting-to-the-remote-node)

- 2) **Running it with a local running copy of the blockchain**
    - **Part 1** [**Hosting your own graftnoded and blockchain**](#hosting-your-own-graftnoded-and-blockchain)
    - **Part 2** [**Connecting a New Wallet or restoring an existing wallet to your own graftnoded**](#connecting-a-new-wallet-or-restoring-an-existing-wallet-to-your-own-graftnoded)

*In this guide, I will show you both ways.*

#### Either way, you need to do this. [**Updating our packages**](#updating-our-packages) AND [**Downloading the binaries**](#downloading-the-binaries)

Optional:

*Creating a folder for your wallet files* [**Creating a folder for your wallet files**](#creating-a-folder-for-your-wallet-files)

I Also, will show you how to stake your Supernode once you have your wallet running [**Staking your Supernode**](#staking-your-supernode)

**Note it is recommended to not use the *root* user when working with Linux machines, especially if it concerns financial applications**

Please consult the hardening guide in the USEFUL LINKS & RESOURCES section on how to setup a non-root user. This is not essential.

- [**USEFUL LINKS & RESOURCES**](#useful-links--resources)

Getting ready:

- Any Ubuntu 18.04 machine is sufficient. Ubuntu server or Ubuntu Desktop will work just fine.

### Getting your SN details:

- Secondly if you are planning on a staking a supernode (If not ignore this and the staking supernode section), you need the details required for staking which you can get from your Supernode, if you are having your Supernode hosted then the URL would be provided to you, if will look something like below:

http://122.78.15.123:18690/dapi/v2.0/cryptonode/getwalletaddress

You can hit this from a web browser and it will return something similar to the below (below has been formatted to fit on the page, usually is in the same line), values have been changed for privacy reasons:

{"testnet":false,"wallet_public_address":"GCdAJoNiAtE9DTQ3UkdU4k3QxeUkbXrKUSECksrL3BuchT4hb1mEkfkfKcoNWdPp1icjDWk3jhjKgh2",
"id_key":"608e8d854a2eb902138b74607d1e0108a7a6c7e903095ab6d984d20c3834324f",
"signature":"e6ec3a907acf1684dcaa3b31b768ca65f0f605c246833b5141ddb1ad21321l80a8566bf01be8f58b8da7704280539d288679260c09d44a576b9234e53b2f2df01"}

Record this for use later

## Updating our packages

````
sudo apt update && sudo apt upgrade -y
````

## Downloading the binaries

- Step 1 : Downloading the binaries and unzipping.
````
cd
wget -c https://github.com/graft-project/GraftNetwork/releases/download/v1.7.4/GraftNetwork_1.7.4.ubuntu-18.04-x64.tar.gz -O - | tar xvz -C .
````
- Step 2 : accessing the folder you just downloaded and unzipped. Check with "ls" to confirm name of file.
````
cd GraftNetwork_1.7.4
````
## Creating a folder for your wallet files

Step 3 : I would suggest you create a seperate directory for you wallet like below:
````
mkdir -p ~/graft-wallets
````
Step 4 : copy graft-wallet-cli to the newly created directory.
````
cp ~/GraftNetwork_1.7.4/graft-wallet-cli ~/graft-wallets/graft-wallet-cli
````
now go to that directory by using "cd"
````
cd ~/graft-wallets
````
## Creating a New Wallet or restoring an existing wallet and connecting to the remote node

**Using a remote node : The quickest way, but not the most secure option, only use with a trusted remote node**

***Ensure you are in the directory where graft-wallet-cli exists*** To check the contents of a drectory use "ls"
- Step 1: Creating a new wallet
````
./graft-wallet-cli --daemon-host graft.community
````
- Step 2 : Restoring an existing wallet
````
./graft-wallet-cli --daemon-host graft.community --restore-deterministic-wallet
````
- Follow the prompts

# NB NB NB Remember to back up your SEED and keep it safe. just type seed in the wallet and follow the prompts.

## Hosting your own graftnoded and blockchain 

**The slower initial setup but more secure way, ensure you have the required disk space for this**

At the time of writing the GRAFT Blockchain is approx: 25GB

Step 1a : Starting graftnoded and partially syncing the blockhain, in order to download the blockchain from a trusted source
````
./graftnoded
````
Leave it for a couple minutes until you see it has synced some blocks. Stop graftnoded by using either "cntrl + c" or typing exit.

- Step 1ab : Now you can download the blockchain to speed up the process. Perform the below commands in order, wait for the curl command to complete as it is the download portion:
````
cd $HOME/.graft/
ls -la
rm -r lmdb
curl http://graftbuilds-ohio.s3.amazonaws.com/lmdb.tar.gz | tar xzf -
cd lmdb && rm em* && rm lo*
cd ~/GraftNetwork_1.7.4
````
Now you can start graftnoded again but this time we will launch it in detached mode so it runs in the background
````
./graftnoded --detach
````
- Step 1b : If you dont want to download the blockchain just run the above command "./graftnoded --detach" from the "~/GraftNetwork_1.7.4.ubuntu-18.04-x64" directory and let it sync fully.

- Step 4 : Checking graftnoded's status.
````
./graftnoded status
````

Once fully synced you can continue to the Creating a New Wallet section and complete that and the staking your supernode section below. [**Connecting New Wallet or restoring an existing wallet to your own graftnoded**](#connecting-new-wallet-or-restoring-an-existing-wallet-to-your-own-graftnoded)

Note for further commands for graftnoded you can run some thing like the below.
````
./graftnoded help
````
For launch flags for graftnoded
````
./graftnoded --help
````
## Connecting a New Wallet or restoring an existing wallet to your own graftnoded

***Ensure you are in the directory where graft-wallet-cli exists*** To check the contents of a drectory use "ls"
- Step 1: Go back to wallet directory we created earlier
````
cd ~/graft-wallets
````
- Step 2: Creating a new wallet
````
./graft-wallet-cli
````
- Step 3 : Restoring an existing wallet
````
./graft-wallet-cli --restore-deterministic-wallet
````
- Follow the prompts

# NB NB NB Remember to back up your SEED and keep it safe. just type seed in the wallet and follow the prompts.

For extra commands available just type help and press enter inside the wallet.

## Staking your Supernode

- Step 1: Ensure address of wallet is same as <wallet_public_address> in the response from your SN that you recorded earlier, see here: [***Getting your SN details***](#getting-your-sn-details)
````
address
````
- Step 2 : Firstly ensure your wallet is reflecting the funds needed once you have you wallet open:
````
balance
````
- Step 3 : If your funds are present, use the details you recorded earlier from the response from your SN (Ensure the wallet you are staking from has the same address as the <wallet_public_address>):
````
stake_transfer <wallet_public_address> <STAKE_AMOUNT> <LOCK_BLOCKS_COUNT> <id_key> <signature>
````

So for the example used in the beginning of the guide, the staking command would look like the below to stake a T1 for 100 blocks = approx 3 hours 20 minutes, (PLEASE DO NOT USE THE BELOW IT IS JUST AN EXAMPLE):
````
stake_transfer GCdAJoNiAtE9DTQ3UkdU4k3QxeUkbXrKUSECksrL3BuchT4hb1mEkfkfKcoNWdPp1icjDWk3jhjKgh2 50000 100 608e8d854a2eb902138b74607d1e0108a7a6c7e903095ab6d984d20c3834324f e6ec3a907acf1684dcaa3b31b768ca65f0f605c246833b5141ddb1ad21321l80a8566bf01be8f58b8da7704280539d288679260c09d44a576b9234e53b2f2df01
````

Note the current maximum LOCK_BLOCKS_COUNT is equal to 5000, this is approximately a week. make sure you know what you are doing before actioning this number.

# Additional content on graft-wallet-cli

#### Graft wallet commands:

In the directory/folder you would like to/or have stored the wallet in (From earlier in the guide this was ~/graft-wallets):
````
./graft-wallet-cli
````
##### Follow prompts to launch and create a new wallet or use an existing wallet in the folder.

To restore an existing wallet from the mmemonic seed:
````
./graft-wallet-cli --restore-deterministic-wallet
````
#### Follow prompts and insert seed when requested.

In wallet commands:

All shown < and > should not be used in the related commands

To get current balance:
````
balance
````
Show the incoming and out-going transactions to this wallet:
````
show_transfers
````
Make a payment:
````
transfer <receiver_wallet_address> <amount>
````
Stake transfer
````
stake_transfer <SUPERNODE_WALLET_PULIC_ADDRESS> <STAKE_AMOUNT> <LOCK_BLOCKS_COUNT> <SUPERNODE_PUBLIC_ID_KEY> <SUPERNODE_SIGNATURE>
````

- Getting your seed from the cli wallet

Once logged into  the wallet and it is synced etc. You just need to type "seed" and press enter, then put you password that you used on creation or restore and follow the prompts, please store this safely as it provides anybody who obtains it the ability to access your funds and send it wherever they like.

Deleting your wallet once you are done staking. Navigate to the folder directory which you launched the wallet to create it or restore it from your seed.

Once done, do as follows:
````
ls
````
This will list the files present in the directory, you should find at 3 files inside that directory, 1 with the exact name that you gave your wallet, another file with the name of the wallet + ".address.txt" and last the name of the wallet + ".keys".

For this example lets consider that we named our wallet "stake-wallet" and we created a new directory before creating/restoring our wallet in the directory called "~/graft-wallets" in our home directory ie. ~/. you can navigate directly to home directory by just doing "cd" and pressing enter.
````
cd ~/graft-wallets
````
````
ls
````
returns
````
stake-wallet stake-wallet.address.txt stake-wallet.keys
````
To delete the files just use the rm command, ENSURE you have the seed stored safely so you can restore the wallet at a later time.
````
rm stake-wallet
rm stake-wallet.address.txt
rm stake-wallet.keys
````

Written by:

#### Github user: Fez29
#### Telegram Handle: @Fezz27

# USEFUL LINKS & RESOURCES

- [Graft Community Guide to deploying a Supernode](https://github.com/graft-community/docs/blob/master/Graft_Supernode_Mainnet_Simple-step-by-step-setup-instructions-for-non-Linux-users_v1.7.md)

- [Graft Community Deb packages, systemd and logrotate setup and Basic Server hardening steps, including operating graftnoded and graft-wallet-cli, troubleshooting supernode and using TMUX](https://github.com/graft-community/docs/blob/master/MaintenanceandServerHardeningguideforGraftSupernode_MultiSN_CommunityDebpackages.md)

- [Graft Network Official Build](https://github.com/graft-project/GraftNetwork/releases)
