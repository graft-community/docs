# Maintenance and Server Hardening guide for Graft Supernode | Multi-SN, Simple Graft commands and troubleshooting - Community Deb packages

-   A big thanks goes out to ***@Jagerman42*** (Telegram Handle) for his contribution to Graft and always supporting the community.

-   A big thanks goes out to ***@el_duderino_007*** (Telegram Handle) for the time and effort put into creating the dummies guide and supporting the community in his role as Community Lead.

-   In order to follow this guide please install nano if it does not exist on your system already or use your preferred text editor, vi for example. 'sudo apt install nano'

-   This guide assumes the name of the user performing the actions is "graft" please change at your discretion to the user your are using on your system. If you genuinely have no clue what user you are logged in as, run:  **echo $USER** or **whoami** and this will reflect the user you are using.

-   For the purpose of the guide we also assume the folder directories where we run supernode from is sn1, sn2, sn3 etc, this can be anything you like as long as each directory is unique for each Supernode. 

-   We also include a logs directory inside of each Supernode directory, once again this is completely up to the user. 

-   We will assume that the user has a general understanding of how to operate the graftnoded and supernode applications, Linux distros and staking their supernode, if not please refer to the guides at the bottom of this document and other mater in the Graft Community Docs repo.

- Take note that if you are upgrading from GraftNetwork version 1.7.* to 1.8.* that your current blockchain will need to be migrated from V1 to V2 so if it appears that graftnoded is hanging after starting 1.8.* it is most like bust with the migration. 

-   Links to some useful content used in building this guide.

### [Graft Community Documentation](https://github.com/graft-community/docs)

-   This guide can be used for testnet with the only difference being that **graftnoded** and **graft-wallet-cli** commands are used with the **--testnet** switch. 
	- Like: graftnoded --testnet

## Introduction to this guide and the tools included and terms used

**Terms used**

- 1) root user : a power/admin user in Linux that is able to perform any command/function on your system and can present huge security risks.
- 2) sudo group : a group of admin users that can perform admin commands/functions but require a password to be passed to complete the request.
    - 2a) Usage : Add sudo in front of any command that requires admin rights in order to execute it successfully. 
- 3) ufw - Ultra Simple Firewall : This allows us to limit connections "into" our server by specific ports. 
    - 3a) Once enable and active. All ports you "allow", remote machines will be able to make "inbound" connections to our server through those ports and the rest of the ports will be blocked.
- 4) updating packages with apt : APT(Advanced Package Tool) is a command line tool that is used for easy interaction with the dpkg packaging system and it is the most efficient and preferred way of managing software from the command line for Debian and Debian based Linux distributions like Ubuntu . It manages dependencies effectively, maintains large configuration files and properly handles upgrades and downgrades to ensure system stability. See here: [**A Beginners Guide to using apt-get commands in Linux(Ubuntu)**](https://codeburst.io/a-beginners-guide-to-using-apt-get-commands-in-linux-ubuntu-d5f102a56fc4)
- 5) ssh : the way we connect to our ubuntu server from a remote machine using putty, powershell or lunux terminal. For an in depth explanation and some usage methods, see here: [Definition: Secure Shell (SSH)](https://searchsecurity.techtarget.com/definition/Secure-Shell)
- 6) systemd - A way to automate and manage services/processes and ensure the configured services are started on boot and restarted if the fail/die. This is not all that systemd does but is what it is used for in this instance. [**Understanding Systemd Units and Unit Files**](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files)
- 7) logrotate - is designed to ease administration of systems that generate large numbers of log files. It allows automatic rotation, compression, removal, and mailing of log files. Each log file may be handled daily, weekly, monthly, or when it grows too large. [**Linux man page**](https://linux.die.net/man/8/logrotate)
- 8) scp - scp copies files between hosts on a network.  It uses ssh(1) for data transfer, and uses the same authentication and provides the same security as ssh(1). scp will ask for passwords or passphrases if they are needed for authentication.[**Linux scp command**](https://www.computerhope.com/unix/scp.htm)

# Index and quick links:

- [**Adding a non-root user**](#add-a-non-root-user)

	In this section we add a non-root user **graft** and add that user to the "sudo" group. This is recommended action and not a must.

- [**Hardening our server**](#hardening-our-server)
	
	In this section there are a few rudimentary hardening steps and tips to ensuring you server is not a easy target. Please BEWARE of locking yourself out of your server after applying 1 or more of these steps.

- [**Setting up folder structures**](#setting-up-our-folder-structure)
	
	In this section we setup a dummy example for a folder structure for having multiple supernodes on the same machine.

- [**Systemd configuration**](#systemd-configuration)
	
	In this section we setup systemd for our graftnoded and supernode services, please make sure that you replace the "graft" value in any path to the user you are logged in as.

- [**Configuring logrotate**](#configuring-logrotate)
	
	In this section we configure logrotate to manage logs in for our graftnoded and supernode services. Note once again: Please replace the "graft" value in any path to the user you are logged in as.

- [**A simple guide to operating and using graftnoded**](#operating-and-using-graftnoded)
	
    In this section the basic usage of graftnoded, graft-wallet-cli and supernode is explained.

- [**Running processes in the background if you have not setup systemd**](#running-processes-in-the-background-if-you-have-not-setup-systemd)
	
	In this section it is explained how to run a process in the background without killing it when closing your terminal session via Putty for example.

- [**TMUX explained**](#tmux)
	
	In this section a few commands for general usage of tmux are shown, please refer to the Useful resources section for a link to a more comprehensive commands list.

- [**Checking if your supernode is running as expected**](#checking-if-your-supernode-is-running-as-expected)
	
	In this section a couple commands to quickly check if the supernode service is running and providing us a supernode list back are shown.

- [**Checking logs**](#checking-logs)
	
	In this section, it is shown how to quickly inspect a log file.

- [**General Troubleshooting of your Supernode setup**](#general-troubleshooting-of-your-supernode-setup)
	
	In this section I have listed my own personal checks that I have performed since the days of public alpha, which have served me well in ensuring that I am able to quickly get my supernode up and running.

- [**Upgrading GraftNetwork and Supernode**](#upgrading)

	A brief explanation on upgrading to latest binaries.

- [**USEFUL LINKS & RESOURCES**](#useful-links--resources)

- This guide has been adjusted to target users using the GraftNetwork official binaries.

### [GraftNetwork](https://github.com/graft-project/GraftNetwork)

## Add a non-root user
Logged in as root and "**graft** being your username you have chosen (the username can be anything)":
````bash
adduser graft
````
* Set and confirm the new user's password at the prompt. A strong password is highly recommended!
````
Set password prompts:
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Note: if you are running as root please exclude any sudo references below:
````
* Follow the prompts to set the new user's information. It is fine to accept the defaults to leave all of this information blank.
````
User information prompts:
Changing the user information for username
Enter the new value, or press ENTER for the default
    Full Name []:
    Room Number []:
    Work Phone []:
    Home Phone []:
    Other []:
Is the information correct? [Y/n]
````
````
usermod -aG sudo graft
````
By default, on Ubuntu, members of the sudo group have sudo privileges.
* Use the su command to switch to the new user account.
````
su - graft
````
* As the new user, verify that you can use sudo by prepending "sudo" to the command that you want to run with superuser privileges. For example we use **apt update** below.
````
username# sudo apt update
````
* For example, you can list the contents of the /root directory, which is normally only accessible to the root user.
````
username#  sudo ls -la /root
````
* The first time you use sudo in a session, you will be prompted for the password of the user account. Enter the password to proceed.
````
Output:
[sudo] password for username:
````
If your user is in the proper group and you entered the password correctly, the command that you issued with sudo should run with root privileges.

# Hardening our server

- The simplest and most effective way to keep our server safe is keep unnecessary software on our server to a minimum and and to always stay up to date with new packages.
````
sudo apt update && sudo apt upgrade -y
````
````
sudo apt autoremove
````
### Enabling ufw:
````
sudo apt install ufw -y
````
**Opening the neccessary ports:**

- SSH:
```
sudo ufw allow 22
```
- Graftnoded port:
````
sudo ufw allow 18980
````
- Enabling UFW (***BEWARE OF LOCKING YOURSELF OUT HERE***) ensure your SSH port is opened:
````
sudo ufw enable
````

### Configuring SSH
This assumes you are connecting to you machine via SSH but have used a password to login, lets enable login via a non-root user with no password. Again we will assume the user is "graft" that is being used on the local machine.

**For a pretty comprehensive guide on SSH best practices see the "Digital Ocean Guide for Systemd management of services" in the [**USEFUL LINKS & RESOURCES**](#useful-links--resources)**

- Press enter to accept defaults unless you already have a SSH key present on your machine and that makes this step unnecessary
```
ssh-key-gen 2048
```
- Press: ENTER
```
ssh-copy-id graft@<remote-machine-ip>
```
- Enter password.
- Now you can use:
Note: All shown < and > should not be used in the related commands.
```
ssh  graft@<remote-machine-ip>
```
to Login with no password.

Next step is disable root logins via ssh, obviously ensure you have created another user to ensure you dont lock yourself out. 
- This step is general SSH best practice as every Linux/Unix distro uses root as the admin user and therefore is an easy target for brute force attacks. 
- Another good idea is to change the port to a custom port for ssh and use the switch "-p <port-number>" with the ssh command.
- Once again ensure you open the port on ufw if enabled to ensure you dont lock yourself out and you have a non-root user configured.

```
sudo nano /etc/ssh/sshd_config
```
- Then uncomment the below line and change value to "no"

````
PermitRootLogin no
````
- To make this take effect immediately:

	- Restart the SSH daemon to apply changes by issuing one of the below commands, specific to your Linux distribution.

```
sudo systemctl restart sshd
sudo service sshd restart
sudo /etc/init.d/ssh restart
```
- Now the root user cannot login via SSH.
- You can change to the root user at anytime by using the below command:
```
su -
```
and inputting the password.

- Changing the port is inside the same file as PermitRootLogin:
```
nano /etc/ssh/sshd_config
```
- Uncomment the Port=22 line and input your custom port instead of 22.
	- Again restart SSH daemon to make it take effect, beware as this could kick you off and lock you out, ***ENSURE PORT IS OPEN ON UFW!***

## Downloading the latest binaries and unzipping them into our home folder.
### GraftNetwork
#### Installing package in order to run graftnoded
````
sudo apt update && sudo apt upgrade && sudo apt install libnorm1
````
Note: The link below will change as newer releases are published, please find the latest release of GraftNetwork (graftnoded, graft-wallet-cli etc) at [GraftNetwork Releases](https://github.com/graft-project/GraftNetwork/releases)

````
sudo apt install wget
````
````
cd && wget https://github.com/graft-project/GraftNetwork/releases/download/v1.8.1/GraftNetwork_1.8.1-ubuntu-18.04-x64.tar.gz
````
- When we untar these files we use the file name that is shown when we pass 'ls' in the terminal. It will be the last section of the above link we used in the download.

Now we check for the file that we downloaded
````
ls
````
- We should see "GraftNetwork_1.8.1-ubuntu-18.04-x64.tar.gz" which is the downloaded file.
- Now we decompress the file.
````
tar -zxvf GraftNetwork_1.8.1-ubuntu-18.04-x64.tar.gz
````
- Now we can delete the compressed file
````
rm GraftNetwork_1.8.1-ubuntu-18.04-x64.tar.gz
````
- Now we cd into the directory after we decompressed the downloaded file.
````
cd GraftNetwork_1.8.1
````
- Now lets check what the contents of the file looks like.
````
ls
````
- The below shows the general contents of what this file will look like.
````
graft-blockchain-export     graftnoded                  graft-wallet-rpc
graft-blockchain-ancestry   graft-blockchain-import     graft-wallet-cli
graft-blockchain-blackball  graft-blockchain-usage      graft-blockchain-depth
graft-gen-trusted-multisig
````
- Ok now lets start setting up so we can our lives easier in running a supernode. Change 'GraftNetwork_1.8.1-ubuntu-18.04-x64' to align with what you downloaded and decompressed in prior steps.
- Below we create a link to the binaries folder so that we can constantly use the same commands even after we upgrade and have a different folder directory.
#### Hard linking GraftNetwork
````
cd
ln -snf GraftNetwork_1.8.1 graftnetwork
````
- Now if you run 'ls' in the terminal you will see a "Folder" like graftnetwork now shows in the home directory.
- This will alow us to access the graft binaries with cd ~/graftnetwork and allow us to form some sort of order when it comes to upgrading to later releases which we will address later in the guide.

- Now we are ready to go with GraftNetwork.
- Now lets setup supernode in the same way with a view to automate everything or at least to make our lives easier.

### Supernode

````
cd && wget https://github.com/graft-project/graft-ng/releases/download/v1.0.4/supernode.1.0.4.ubuntu-18.04.x64.tar.gz
````
- When we untar these files we use the file name that is shown when we pass 'ls' in the terminal. It will be the last section of the above link we used in the download.
Now we check for the file that we downloaded
````
ls
````
- We should see "supernode.1.0.4.ubuntu-18.04.x64.tar.gz" which is the downloaded file.
- Now we decompress the file.
````
tar -zxvf supernode.1.0.4.ubuntu-18.04.x64.tar.gz
````
- Now we can delete the compressed file
````
rm supernode.1.0.4.ubuntu-18.04.x64.tar.gz
````
- Now we run ls and cd into the directory that is left after we decompressed the downloaded file.
````
cd supernode.1.0.4.ubuntu-18.04.x64
````
- Now lets check what the contents of the file looks like.
````
ls
````
- The below shows the general contents of what this file will look like.
````
config.ini  graftlets  supernode
````
Now we setup a hardlink to our supernode download folder called graftsupernode, which will make upgrading in future much easier and quicker.

#### Hard linking Supernode

````
ln -snf supernode.1.0.4.ubuntu-18.04.x64 graftsupernode
````

## Setting up our folder structure for Multiple Supernodes

- **Copy graftsupernode contents and create sn directories**
Note if you already have existing directories do not do the below,

- Section 1 - Setting up from scratch
````
cp -r ~/graftsupernode/config.ini ~/sn1/config.ini

cp -r ~/graftsupernode/config.ini ~/sn2/config.ini

cp -r ~/graftsupernode/config.ini ~/sn3/config.ini
````
- **Create Our logs Folders**

````
mkdir -p ~/sn1/logs
````
````
mkdir -p ~/sn2/logs
````
````
mkdir -p ~/sn3/logs
````

- **Edit config.ini files** Repeat for all sn folders created.

````
nano ~/sn1/config.ini
````

- **The Following values are the important values to change when running multi-sn and should be unique per Supernode:**

````
http-address=0.0.0.0:18690
````

````
data-dir=
````

- Change the port on ***http-address=***

- **Change the directory according to the folders you created in the first step for ***data-dir=*** for each respective config.ini in each directory. (Replace value after /home/<value> with your user)**
- **Something like:**

````
data-dir=/home/graft/sn1/
````

````
data-dir=/home/graft/sn2/
````

- **Obviously as per all other guides instructions, add your wallet into config.ini that you will be using to stake with:**
Note: All shown < and > should not be used in the related commands.
````
	wallet-public-address=<Your_wallet_address>
````

- **Start supernode and CHECK that supernode.keys is inside your directory for each sn. Refer to below where we setup systemd to launch each Supernode.**


## Systemd configuration

## Create systemd service for graftnoded:

````
sudo nano /etc/systemd/system/graftnoded.service
````

- Input below text into the new file <change graft (Inside the User field and after /home/ in the ExecStart field) to your user (find this out by using "whoami" command or "echo $USER")>:

````
[Unit]
Description=Graft node Mainnet
After=network-online.target

[Service]
User=graft
Type=simple
WorkingDirectory=~
Restart=always
ExecStart=/home/graft/graftnetwork/graftnoded --non-interactive --log-level 1
#LimitNOFILE=8192
Environment=TERM=xterm

[Install]
WantedBy=multi-user.target
````
- Cntrl + x to save

## Enable graftnoded.service to automatically start graftnoded after boot.

````
sudo systemctl enable graftnoded.service
````

- The output will advise you of a symlink being created something like below:
````
Created symlink /etc/systemd/system/multi-user.target.wants/graftnoded.service → /etc/systemd/system/graftnoded.service.
````

## Start Graftnoded
````
sudo systemctl start graftnoded.service
````
To view the systemd logs for your newly created graftnoded.service:

````
sudo journalctl -u graftnoded.service -af
````

## Creating the graft-supernode.service file:
````
sudo nano /etc/systemd/system/graft-supernode@.service
````

- Input below text into the new file <change graft (Inside the User field and in WorkingDirectory and after /home/ in the ExecStart field) to your user (find this out by using "whoami" command or "echo $USER")>:

Note: If you use the exact below systemd file, ensure you did create the log file directories as the linked section here [**Supernode**](#supernode)

````
[Unit]
Description=Graft supernode
After=graftnoded.service

[Service]
User=graft
Type=simple
WorkingDirectory=/home/graft/%i
Restart=always
ExecStart=/home/graft/graftsupernode/supernode --config-file config.ini --log-file /home/graft/%i/logs/%i.log --log-level 1
Environment=TERM=xterm
#LimitNOFILE=8192

[Install]
WantedBy=multi-user.target
````
Cntrl + x to save

In the above
	
   - The "%i" is a variable which is defined when we run the enable command below. the input between @ and . is input as a variable into our file on creation and the same goes for when we start the service. 

In the below example: 
````    
sn1 would be our variable in the first instance. 
````
````
sn2 would be the variable in the second instance.
````

## Enable graft-supernode@.service to automatically start graft-supernode after boot and to only start if graftnoded is up and running.

- sn1

````
sudo systemctl enable graft-supernode@sn1.service
````
- sn2

````
sudo systemctl enable graft-supernode@sn2.service
````

## Starting graft-supernode with systemd

- sn1

````
sudo systemctl start graft-supernode@sn1.service
````

- sn2

````
sudo systemctl start graft-supernode@sn2.service
````

Now we can also stop the graft-supernode service with the below:
- sn1

````
sudo systemctl stop graft-supernode@sn1.service
````

To view the systemd logs for your newly created graft-supernode@sn1.service:

````
sudo journalctl -u graft-supernode@sn1.service -af
````
And you can do the same but for sn2:
````
sudo journalctl -u graft-supernode@sn2.service -af
````

## Configuring logrotate
#### graftnoded logs
````
sudo nano /etc/logrotate.d/graftnoded-logs
````
- Input below text into the new file:
````
/home/graft/.graft/graft.log {
    daily
    rotate 5
    missingok
    compress
    create
}
````
#### supernode logs

Remember to change graft to your user (find this out by using "whoami" command)
````
sudo nano /etc/logrotate.d/sn1-logs
````
- Input below text into the new file:
````
/home/graft/sn1/logs/sn1.log {
    daily
    rotate 5
    missingok
    compress
    create
}
````
- Cntrl + x to save
````
sudo nano /etc/logrotate.d/sn2-logs
````
- Input below text into the new file:
````
/home/graft/sn2/logs/sn2.log {
    daily
    rotate 5
    missingok
    compress
    create
}
````
- Cntrl + x to save

- To check that logrotate is actually working we can run the below command:
````
sudo /usr/sbin/logrotate -f /etc/logrotate.conf
````
- Then check the log file directory for a compressed version of the log-file.
- We then check the chron job to ensure that force is enabled so we are assured the log-files will be compressed and rotated daily.
	- Check the logrotate chron script exists
````
sudo cat /etc/cron.daily/logrotate
````
- If it exists it will return something like:
````
#!/bin/sh

# skip in favour of systemd timer
if [ -d /run/systemd/system ]; then
    exit 0
fi

# this cronjob persists removals (but not purges)
if [ ! -x /usr/sbin/logrotate ]; then
    exit 0
fi

/usr/sbin/logrotate /etc/logrotate.conf
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit $EXITVALUE

````
- Then check the directory where the log file exists you should see another file now which is a result of logrotate compressing the .log file.

## Port forwarding/"exposing the port" is required to the graftnoded port as of the time of writing to ensure that your graftnoded can communicate with other machines on the network and so your supernode appear as active on the GraftRTABot on Telegram.

Port for mainnet graftnoded at the time of writing is : ***18980***

## Operating and using graftnoded

Please note that if you are running compiled binaries or downloaded the binaries, all commands should be run in the same folder as the binary resides/lives and with ./ in front of the mentioned command, eg, ./graftnoded status.

#### Mainnet:
````
Default Port for graftnoded: 18980
````
#### Public Testnet:
````
Default Port for graftnoded: 28880
````
#### Launch graftnoded (Here we assume you have followed the previous part of the guide setting up GraftNetwork, refer to below link for section):
- [**GraftNetwork**](#graftnetwork-1)
````
~/graftnetwork/graftnoded --detach
````
Get status of graftnoded:
````
~/graftnetwork/graftnoded status
````
Get graftnoded launch options:
````
~/graftnetwork/graftnoded --help
````
Get graftnoded interactive commands like status previously mentioned:
````
~/graftnetwork/graftnoded help
````
#### If you did not follow the setup of graftnetwork link as referred to in above section, you need to run these commands in the directory/folder where the binaries live and put ./ in front like:
````
./graftnoded --detach
````
#### Graft wallet commands:

In the directory/folder you would like to store the wallet in:
````
~/graftnetwork/graft-wallet-cli
````
##### Follow prompts to launch and create a new wallet or use an existing wallet in the folder.

To restore an existing wallet from the mmemonic seed:
````
~/graftnetwork/graft-wallet-cli --restore-deterministic-wallet
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
Once logged into  the wallet and it is synced etc. You just need to type "seed" and press enter, then put you password that you used on creation or restore and follow the prompts, please store this safely as it provides anybody who obtains it the ability to access your funds and send it wherever they like.

Deleting your wallet once you are done staking. Navigate to the folder directory which you launched the wallet to create it or restore it from your seed.

Once done, do as follows:
````
ls
````

- Getting your seed from the cli wallet

This will list the files present in the directory, you should find at 3 files inside that directory, 1 with the exact name that you gave your wallet, another file with the name of the wallet + ".address.txt" and last the name of the wallet + ".keys".

For this example lets consider that we named our wallet "stake-wallet" and we created a new directory before creating/restoring our wallet in the directory called "wallets" in our home directory ie. ~/. you can navigate directly to home directory by just doing "cd" and pressing enter.
````
cd ~/wallets
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

### Running processes in the background if you have not setup systemd

You have an option of either TMUX or Screen, I am sure there are other options but these are the two that are popular. I myself have used TMUX and found it prety easy to use and powerful, therefore I will explain the usage here, I will post links at the bottom of the guide to further instructions/cheat sheets for tmux and also 1 for screen which I will not include in this guide as my usage of screen is extremely limited.

#### TMUX

Firstly Install tmux on your machine.
````
sudo apt update && sudo apt upgrade -y
````
Once complete you generate a session, in this example we will call ours "graftsupernode_tmux":
````
tmux new -s graftsupernode_tmux
````
This will launch a tmux session and have the first window open, this is window number 0.
	- The Tmux session would have started in the directory that you launched it from, ie. If you were in ~/ directory (home directory) that is where the session will be.
	- Now you can run any commands you like such as the supernode launch command for example.
	- The difference is now if you detach from the "graftsupernode_tmux" session, your process will not stop.

In order to detach from the tmux session with a default tmux configuration setup.

Press:
````
Cntrl + b
````
Then press:
````
d
````
This will now detach the tmux session and you will back to where you started before you launched the "graftsupernode_tmux" session.

In order to re-attach to the same session we created earlier we use the below command:
````
tmux a -t graftsupernode_tmux
````
This will return us back to the session we created earlier.

Now lets open a second window:
Press:
````
Cntrl + b
````
Then press:
````
c
````
This will open window number 1.
Now you have a second "terminal" window in tmux to work in.

If you would like to return to the previous "terminal" window.
Press:
````
Cntrl + b
````
Then press:
````
0
````

If you would like to exit/close a window, just type exit and press enter, this will close the window you are in.

This should be enough to allow you a general understanding of tmux and its usage, please refer to the Resources section at the bottom of this guide where I have posted some useful links to find more complete command lists, or google it yourself as there is many articles on both tmux and screen usage. 

#### Checking if your supernode is running as expected

Locally on the machine supernode is running on (run in terminal on the local machine):
````
curl 127.0.0.1:28690/debug/supernode_list/1
````
From external (from your home pc to your VPS for example, insert in browser):
Note: All shown < and > should not be used in the related commands
````
http://<Your_Server_IP>:28690/debug/supernode_list/1
````
The above commands show all supernodes that your supernode has picked up, in order to show all supernodes your supernodes recognizes as active please replace 1 with 0 at the end,

Like:
````
curl Your_Server_IP:28690/debug/supernode_list/0
````
#### Checking logs
CHecking graftnoded logs
````
tail -f -n 150 ~/.graft/graft.log
````

- The 150 above represent the amount of lines to be shown please feel free to adjust ths to your requirements.

Also the path can be adjusted to suit your needs and can be used to check the path you have mapped your supernode to, if you did not map it then the log file will exist in the same location where you ran it.

Copying log files from your remote linux machine to a local linux machine:
Note: All shown < and > should not be used in the related commands.
````
scp -P <remote_port> <remote_user>@<remote_machine_ip>:/<remote_machine_directory> ~/<local_machine_directory>
````
Copying log files from your local linux machine to a remote linux machine:
````
scp -P <remote_port> ~/<local_machine_directory> <remote_user>@<remote_machine_ip>:/<remote_machine_directory>
````
- -P <remote_port> can be left out if you are suing the generic SSH port or define -P 22.

Another useful maintenance command to check disk space is:
```
df -h
```
This will tell you if the disk is full or not.

# General Troubleshooting of your Supernode setup

If your supernode is not communicating with the GraftRTABot which can be used on Telegram, the few belows steps should get you pointing in the right direction:

Step 1:

- Ensure that graftnoded is synced and on the same blockheight as the GraftRTABot.
You can check the blockheight of your Graftnoded by using the status command mentioned previously.

Step 2:

- Ensure you that graftnoftnoded port is exposed/portforwarded as mentioned previously in this guide.
	- Supernode port should be optional to be exposed/portforwarded and is not essential to the working of your supernode, graftnoded port is though as mentioned previously. 
	- Supernode can however be useful to test if port forwarding is working as you can use the curl commands from an external source and see if you get a valid response.

Step 3:
-  Check that the graftnoded port and if desired the supernode port are open on UFW if it is active, How to check UFW status has also been mentioned previously in this guide.

Step 4:

- Check that supernode is actually running and responding to the curl commands mentioned previously in this guide locally and if possible externally. If graftnoded was not running you should know by now by performing step 1.

Step 5:

- Check the RPC port in config.ini and confirm it is aligned with the environment your are trying to use.
	- Mainnet: 18981
	- Testnet: 28881  

If you can tick all these boxes then you can be pretty sure your supernode setup is ok and just give it some time to show on the bot.

# Upgrading

### Upgrading GraftNetwork

- Follow the relevant section in the guide relating to [**GraftNetwork**](#graftnetwork-1) using the latest versions download link. Ensure you perform the last step ([**Hard Linking GraftNetwork**](#hard-linking-graftnetwork)) using the directory from your latest download.

### Upgrading Supernode

- Follow the relevant section in the guide relating to [**Supernode**](#supernode) using the latest versions download link. Ensure you perform the last step ([**Hard Linking Supernode**](#hard-linking-supernode)) using the directory from your latest download.


#### Please send me a telegram message if you have suggestions or feel free to fork the graft community docs and add your additions to this doc, it will be appreciated and welcomed.

#### Telegram Handle: @Fezz27

#### GitHub Page
[**Fez29 Github Repos**](https://github.com/Fez29?tab=repositories)

# USEFUL LINKS & RESOURCES

- [**Digital Ocean Guide for SSH Setup**](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-debian-9)

- [**Definition: Secure Shell (SSH)**](https://searchsecurity.techtarget.com/definition/Secure-Shell)

- [**Digital Ocean Guide for Systemd management of services**](https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units)

- [**Linode using logrotate to manage log files**](https://www.linode.com/docs/uptime/logs/use-logrotate-to-manage-log-files/)

- [**TMUX Cheat Sheet**](https://tmuxcheatsheet.com/) AND [**Linux Tmux Cheat Sheet - computingforgeeks**](https://computingforgeeks.com/linux-tmux-cheat-sheet/)

- [**Screen Guide**](https://linuxize.com/post/how-to-use-linux-screen/)

- [**A Beginners Guide to using apt-get commands in Linux(Ubuntu)**](https://codeburst.io/a-beginners-guide-to-using-apt-get-commands-in-linux-ubuntu-d5f102a56fc4)